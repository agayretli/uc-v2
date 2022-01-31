<template>
  <div>
    <div class="modal" v-if="newUser">
        <a v-on:click="newUser=false" class="close cursor-pointer">✖</a>
        <p>{{newUserMessage}}</p>
    </div>
    <div class="left-panel">
      <user
        v-for="user in users"
        :key="user.userID"
        :user="user"
        :selected="selectedUser === user"
        @select="onSelectUser(user)"
      />
    </div>
  </div>
</template>

<script>
import socket from "../socket";
import User from "./User";

export default {
  name: "List",
  components: { User },
  data() {
    return {
      selectedUser: null,
      users: [],
      newUser:false,
      newUserMessage:''
    };
  },
  methods: {
    onSelectUser(user) {
      this.selectedUser = user;
    },
  },
  created() {
    socket.on("connect", () => {
      this.users.forEach((user) => {
        if (user.self) {
          user.connected = true;
        }
      });
    });

    socket.on("disconnect", () => {
      this.users.forEach((user) => {
        if (user.self) {
          user.connected = false;
        }
      });
    });

    socket.on("users", (users) => {
      users.forEach((user) => {
        for (let i = 0; i < this.users.length; i++) {
          const existingUser = this.users[i];
          if (existingUser.userID === user.userID) {
            existingUser.connected = user.connected;
            return;
          }
        }
        user.self = user.userID === socket.userID;
        this.users.push(user);
      });
      // put the current user first, and sort by username
      this.users.sort((a, b) => {
        if (a.self) return -1;
        if (b.self) return 1;
        if (a.username < b.username) return -1;
        return a.username > b.username ? 1 : 0;
      });
    });

    socket.on("user connected", (user) => {
      this.newUser = true;
      this.newUserMessage = "Connected user's username: " + user.username;
      for (let i = 0; i < this.users.length; i++) {
        const existingUser = this.users[i];
        if (existingUser.userID === user.userID) {
          existingUser.connected = true;
          return;
        }
      }
      this.users.push(user);
    });

    socket.on("user disconnected", (id) => {
      for (let i = 0; i < this.users.length; i++) {
        const user = this.users[i];
        if (user.userID === id) {
          user.connected = false;
          break;
        }
      }
    });
  },
  destroyed() {
    socket.off("connect");
    socket.off("disconnect");
    socket.off("users");
    socket.off("user connected");
    socket.off("user disconnected");
  },
};
</script>

<style scoped>
.left-panel {
  position: fixed;
  left: 0;
  top: 0;
  bottom: 0;
  width: 260px;
  overflow-x: hidden;
  background-color: #3f0e40;
  color: white;
}

.modal{
  z-index: 9999;
  position:absolute;
  left:50%;
  margin-left:-250px;
  top:50%;
  margin-top:-100px;
  background:white;
  width:500px;
  height:200px;
  text-align:center;
  box-shadow: 0px 0px 0px 1920px rgba(0,0,0,0.75);
}

.modal a.close{
  position:absolute;
  right:15px;
  top:10px;
}

.modal p{
  margin:0;
  line-height:200px;
}

.cursor-pointer{
  cursor:pointer;
}
</style>
