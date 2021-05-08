<template>
    <div class=" hidden w-3/5 space-y-4 shadow-md mt-16 fixed vs:flex flex-col ring-1 ring-gray-200 bg-white  text-gray-600 font-bold  px-2 py-2">
    <div class="flex   ">
        <img src="/img/pasaBUYLogoOnly.png" class="w-12 h-10">
        <p class="text-2xl text-red-700 font-extrabold">pasaBUY</p>
    </div>
     <router-link to="/dashboard" class="rounded-2xl   ">  
     <div class="rounded-2xl items-center flex pl-2 h-10 space-x-2 ">
       <span class="material-icons">
        home  
      </span> 
      <span class="text text-gray-500">Home</span>
      </div>
      </router-link>
     <router-link to="/messages"  class="rounded-2xl   "> 
     <div class="rounded-2xl items-center flex h-10 space-x-2 pl-2 ">
      <span class="material-icons cursor-pointer">
        chat
      </span>
      <p class=" text-gray-500">Messages</p>
      </div></router-link>
      <router-link to="/ss"  class="rounded-2xl   "> 
      <div class="rounded-2xl items-center flex h-10 space-x-2 pl-2 ">
       <span class="material-icons ">
            notifications
          </span>
      <p class="text-gray-500">Notifications</p>
      <span class="text-red-500">{{ unreadNotif.length }}</span>  
      </div></router-link>
      <button  @click="setDispatches(userPersonal.email)" class="rounded-2xl">
      <div class="rounded-2xl items-center flex h-10 pl-2 space-x-2 ">
       <span class="material-icons ">
            account_circle
          </span>
      <p class="text-gray-500">Profile</p>   
      </div>
      </button>
      <hr>
      <router-link to="/orders"  class="rounded-2xl  ">
       <div class="rounded-2xl items-center flex h-10 pl-2 space-2-x-4 ">
       <span class="material-icons ">
          shopping_bag
          </span>
      <p class="text-gray-500">Orders</p>   
      </div></router-link>
      <router-link to="/orders"  class="rounded-2xl  ">
       <div class="rounded-2xl items-center flex h-10 space-x-2 pl-2 ">
       <span class="material-icons ">
          delivery_dining
          </span>
      <p class="text-gray-500">Deliveries</p>   
      </div></router-link>
        <router-link to="/deliver" class="rounded-2xl">
       <div class="rounded-2xl items-center flex h-10 space-x-2 pl-2 ">
       <span class="material-icons ">
          list
          </span>
      <p class="text-gray-500">Shopping Lists</p>   
      </div></router-link>
        <router-link to="/shopping-list" class="rounded-2xl  ">
       <div class="rounded-2xl items-center flex h-10 space-x-2 pl-2 ">
       <span class="material-icons ">
         manage_accounts
          </span>
      <p class="text-gray-500">Account Settings</p>   
      </div></router-link>
       <button @click="logout"  class="rounded-2xl ">
       <div class="rounded-2xl items-center flex h-10 space-x-2 pl-2 ">
       <span class="material-icons ">
         logout
          </span>
      <p class="text-gray-500">Log Out</p>   
      </div></button>
      
    </div>
</template>
<style scoped>
.router-link-active{
  color:red;
 background-color: rgba(236, 233, 233, 0.705);
}
</style>
<script>
import store from "../store/index.js"
import api from "../api.js"
export default {
  methods:{
    setDispatches(email) {
      store.dispatch("getUserInfo", email).then(() => {
        store.dispatch("getNotAuthUserAddress", email).then(() => {
          store.dispatch("getUserFollow", email).then(() => {
            this.$router.push({
              name: "Profile",
              query: { ID: this.toEncrypt(email) },
            });
          });
        });
      });
    },
    logout() {
      console.log("logout");
      window.Echo.leave("App.Models.User." + this.user.indexUserAuthentication);
      window.Echo.leave("public.123");
      api.post("api/logout").then(() => {
        sessionStorage.removeItem("vuex");
        sessionStorage.removeItem("isLoggedIn");
        this.$router.push({ name: "Home" });
      });
    },
      toEncrypt(val) {
      return btoa(val);
    },
  },
  
  mounted(){
var roomId = 123
    window.Echo.join(`public.${roomId}`)
    .here((users) => {
    //set the users who's already online before you online
    for(var i=0;i<users.length;i++){
        store.commit('setOnlineUsers',users[i].email)
    }
    console.log(users)
    })
    .joining((user) => {
       //add the user who just logged in tpo online
        store.commit('setOnlineUsers',user.email)
        console.log(user.email, " is online");
    })
    .leaving((user) => {
      //remove users who leave the channel to online
        store.commit('removeFromOnlineUsers',user.email)
        console.log(user.email, " leaved");
    })
    .listen('.post.new', () => {
        //
      console.log("someone posted")
      this.debounceMethodGetPosts();
    })
    .error((error) => {
        console.error(error);
    });

    window.Echo.private(
      "App.Models.User." + this.user.indexUserAuthentication
    ).notification((notification) => {
      console.log("in the navbar", notification.type);
      this.debounceMethod(notification.type);
    });
  },

  computed: {
    unreadNotif() {
      return store.getters.getUnreadNotif.filter((x) => {
        return x.type != "App\\Notifications\\newTransactionNotification";
      });
    },
    unreadNotifChat() {
      return store.getters.getUnreadNotif.filter((x) => {
        return x.type == "App\\Notifications\\newTransactionNotification";
      });
    },
    user() {
      return store.getters.getUser;
    },
    userPersonal() {
      return store.getters.getPersonal;
    },
  }
  }


</script>