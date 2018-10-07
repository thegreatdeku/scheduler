<template>
  <div class="hello">
    <div v-for="member in selected">
      {{ member.name }}
      <div>
        {{ member.score }}
      </div>
    </div>
    <button v-on:click="pickUsers"></button>
  </div>
</template>

<script>
import userData from '../assets/users.json';
export default {
  name: 'Home',
  props: {
    msg: String
  },
  data: () => ({
    users: userData,
    selected: []
  }),
  methods: {
    pickUsers: function() {
      if(localStorage.users) {
        this.users = JSON.parse(localStorage.getItem('users'));
      }
      let selectedUsers = [];
      console.log(this.users);
      // Pick the first 3-5 users off the list that have not been selected yet
      for(let i = 5; i >= 3; i--) {
        if(!this.users[i]) {
            continue;
        }
        if (this.users[i].score == 0) {
            selectedUsers = this.users.slice(0,i);
            break;
        }
      }

      // If everyone has been selected already, pick off the top of the list
      let max = Math.floor(Math.random()*(5-3+1)) + 3;
      if(selectedUsers.length == 0) {
        selectedUsers = this.users.slice(0,max);
      }

      this.selected = selectedUsers;
      // Update the users
      console.log(selectedUsers);
      this.users = this.users.slice((this.users.length - selectedUsers.length)*-1);
      console.log(this.users);
      for (let i = 0; i < selectedUsers.length; i++) {
          selectedUsers[i].score = selectedUsers[i].score + 1;
          this.users.push(selectedUsers[i]);
      }

      // Update localStorage
      localStorage.setItem('users',JSON.stringify(this.users));
      return selectedUsers;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
