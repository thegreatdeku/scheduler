<template>
  <div class="home">
   
    <div v-if="user" class="user-area">
      <div class="subtitle">
        Welcome,
      </div>
      <div class="title">
        {{ user.name }}
      </div>
      <div class="btn-container">
        <div v-if="!hasCoffee" class="msg-coffee">
          You have gone to coffee with all your coworkers! Find a lunch group.
        </div>
        <button v-if="hasCoffee" v-on:click="pickPartner" class="button coffee">Find Coffee Partner</button>
        <button v-on:click="pickUsers" class="button">Find Lunch Group</button>
      </div>
    </div>
    <div v-else>
      <div class="title">
        Welcome
      </div>
      <div class="msg">
        Please enter your name and email to find <br> your coffee partner
        or lunch group!
      </div>
      <div class="new-user">
        <form v-on:submit.prevent="addUser">
        <div class="input-row">
          <input type="text" placeholder="Name" v-model="userName">
        </div>
        <div class="input-row">
          <input type="text" placeholder="Email" v-model="userEmail">
        </div>      
        <button class="button">Add User</button>
        </form>
      </div>
    </div>
    <div v-if="recentGroup.length && showRecent" class="subtitle outing">
      <div v-if="recentGroup.length == 1" class="title">
        Your most recent outing (coffee)
      </div>
      <div v-else>
        Your most recent outing (lunch)
      </div>
    </div>
    <div v-if="selected.length" class="user">
      <div class="name">
        {{ user.name }} (You)
      </div>
    </div>
    <div v-for="member in selected" class="user">
      <div class="name">
        {{ member.name }}
      </div>
      <div v-if="!showLunch" class="msg">
        <div v-if="(member.score == 1)">
          New lunch buddy!
        </div>
        <div v-else>
          {{ member.score - 1 }} time(s) to lunch
        </div>
      </div>
    </div>
    
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
    selected: [],
    user: null,
    userName: "",
    userEmail: "",
    showCoffee: true,
    showLunch: true,
    recentGroup: [],
    hasCoffee: true,
    showRecent: true
  }),
  methods: {
    addUser: function() {
      this.user = {
        name: this.userName,
        userEmail: this.userEmail
      }
      localStorage.setItem('user',JSON.stringify(this.user));
    },
    pickUsers: function() {
      if(localStorage.users) {
        this.users = JSON.parse(localStorage.getItem('users'));
      }
      let selectedUsers = [];
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
      this.users = this.users.slice((this.users.length - selectedUsers.length)*-1);
      for (let i = 0; i < selectedUsers.length; i++) {
          selectedUsers[i].score = selectedUsers[i].score + 1;
          this.users.push(selectedUsers[i]);
      }

      // Update localStorage
      localStorage.setItem('users',JSON.stringify(this.users));
      localStorage.setItem('group',JSON.stringify(this.selected));
      this.showCoffee = true;
      this.showLunch = false;
      this.showRecent = false;

      return selectedUsers;
    },
    pickPartner: function() {
      if(localStorage.users) {
        this.users = JSON.parse(localStorage.getItem('users'));
      }

      // Pick the first person with who has not been selected
      for(let i = 0; i < this.users.length; i ++) {
        if(this.users[i].hasCoffee == 0) {
          this.selected = [this.users[i]];
          this.users[i].hasCoffee = 1;
          localStorage.setItem('users',JSON.stringify(this.users));
          localStorage.setItem('group', JSON.stringify([this.users[i]]));
          localStorage.setItem('partner', 1);
          this.showCoffee = false;
          this.showLunch = true;
          this.showRecent = false;
          return;
        }
      }

      // All users have been selected
      this.selected = [];
      this.showCoffee = false;
      this.showLunch = true;
      this.hasCoffee = false;
      this.showRecent = false;

      localStorage.setItem('partner', -1);
      return; 
    }
  },
  mounted: function() {
    if(localStorage.user) {
      this.user = JSON.parse(localStorage.getItem('user'));
    }
    if(localStorage.partner == -1) {
      this.hasCoffee = false;
    }
    if(localStorage.group) {
      this.recentGroup = JSON.parse(localStorage.getItem('group'));
      this.selected = this.recentGroup;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.msg-coffee {
  margin-bottom: 16px;
}
.btn-container {
  margin: 16px auto;
}
.user-area {
  width: 100%;
}
.user {
  text-align: left;
  margin: 0 auto;
  max-width: 600px;
  border: 1px #aaa solid;
  background-color: #fff;
  border-radius: 3px;
  margin-bottom: 1.25rem;
  padding: 1rem;
}
.name {
  font-size: 20px;
  font-weight: 600;
}
.title,.subtitle {
  font-family: Archer SSm A,Archer SSm B,serif;
  color: #777676;
  font-size: 36px;
}
.subtitle {
  font-size: 24px;
}
.subtitle.outing {
  margin-bottom: 16px;
}
.button {
  -webkit-appearance: none;
    -moz-appearance: none;
    border-radius: 0;
    font-family: -apple-system,BlinkMacSystemFont,Segoe UI,Oxygen,Ubuntu,Cantarell,Fira Sans,Droid Sans,Helvetica Neue,sans-serif;
    font-weight: bolder;
    line-height: normal;
    margin: 0 0 1rem;
    position: relative;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    padding: .5625rem 1.125rem .625rem;
    font-size: 1rem;
    background-color: #69b5b1;
    border: 0 solid #4c9995;
    color: #fff;
    border-radius: 4px;
    transition: background-color .3s ease-out;
    margin-right: 8px;
    cursor: pointer;
}
.button.coffee {
  background-color: 	#9c6f44;
  border: #724e2c;
}
input {
  background-color: #fff;
  border: 1px solid #e6e6e6;
  box-shadow: none;
  display: block;
  font-size: 14px;
  height: 30px;
  width: 240px;
  margin: 10px auto;
  padding: 5px;
}
</style>
