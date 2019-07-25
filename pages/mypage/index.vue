<template>

  <div>
    <div>
      <v-toolbar>
        <v-toolbar-title>マイページ</v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
    </div>
        
    <v-card style="padding-bottom:10px;">
      <button type="button" @click="logout">ログアウト</button>
      <div style="margin-left:10px;padding-top:10px;">
        <v-avatar size="56px" >
          <img
            :src="user.photoURL"
          >
          </v-avatar>
          <a class="notNewLine">
            {{user.displayName}}
          </a>
      </div>
    </v-card>
    
    <v-card style="margin-top:20px;padding-top:10px;padding-bottom:10px;">
    <nuxt-link to="/mypage/item_list"><p style="font-size:1.1em">出品したお土産</p></nuxt-link>
    <nuxt-link to="/mypage/request_list"><p style="font-size:1.1em">申請・取引</p></nuxt-link>
    </v-card>

  </div>
</template>

<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'

export default {
  fetch ({ store, route,redirect }) {
    if (!store.state.user.user) {
      if(route.name != "/login"){
      return redirect('/login')
      }else{
       return redirect('/mypage/')
      }
    }
  },
  data() {
    return {
      user: {},  // ユーザー情報
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
        this.user = user ? user : {}
    })
  },
  methods : {
    ...mapActions(['setUser']), 
          logout() {
        const self = this
        firebase.auth().signOut()
        .then(_ => {
          console.log("ログアウト成功")
          this.$store.dispatch('user/fecthUser',)
          //self.$router.push("/login")
        }).catch((error) => {
          alert(error)
        })
      },
  },
  
};


</script>
<style scoped>
p {
  margin-left: 15px;
  margin-top: 10px;
}
a{
  text-decoration: none !important;
  color:black;
}
/*改行させるかよ*/
.notNewLine{
  text-align:center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>