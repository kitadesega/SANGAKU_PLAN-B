<template>
  <div>
    <v-card>
      <v-avatar
    size="56px" >
        <img
          :src="user.photoURL"
        >
        </v-avatar><a class="notNewLine">
        {{user.displayName}}
      </a>
      <div style="display:flex;">
      <div style="margin-left:auto;">
      <v-btn v-on:click="logout">ログアウト</v-btn>
      </div>
      </div>
    </v-card>
    <v-card style="padding-top:10px;padding-bottom:10px;">
    <!-- <p>いいね！・閲覧履歴</p>
    <p>出品したお土産</p>
    <p>交換した商品</p>
    <p>取引中のお土産</p>
    <nuxt-link to="/mypage/request_list#tab-3"><p>申請中のお土産</p></nuxt-link> -->
    <nuxt-link to="/mypage/request_list"><p>申請・取引</p></nuxt-link>
    </v-card>
    <v-card>
        <p><span class="cyan--text">ガイド・お問い合わせ</span></p>
        <p>ガイド</p>
        <p>お問い合わせ</p>
        <p>Q&A</p>
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
    }
  },
  
};


</script>
<style scoped>
p {
  margin-left: 15px;
  margin-top: 10px;
}

/*改行させるかよ*/
.notNewLine{
  text-align:center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>