<template>
  <div style="margin-top:100px;">
    <v-layout justify-center>
      <div style="margin-left:30px">
        <v-btn color="info" @click="twitterLogin">Twitterでログイン</v-btn>
        <v-spacer />
        <v-btn color="error" @click="googleLogin">Googleでログイン</v-btn>
      </div>
    </v-layout>
  </div>
</template>

<script>
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'

export default {
  //ログインしていたらマイページにリダイレクト
  fetch ({ store, route,redirect }) {
    if (store.state.user.user) {
      return redirect('/mypage')
    }
  },
  mounted() {
    firebase.auth().onAuthStateChanged(user => {
      if (user) {
        console.log(user.uid);
        // User is signed in.
        this.$store.dispatch('user/fecthUser', "id" )
        this.$router.push("/mypage")
        } else {
          this.$router.push("/login")
        }
        //firestore設定
    });
    
  },
  methods : {
    ...mapActions(['setUser']),

    twitterLogin () {
      var provider = new firebase.auth.TwitterAuthProvider()
      firebase.auth().signInWithPopup(provider)
      firebase.auth().onAuthStateChanged(user => {
        if (user) {
          this.$store.dispatch('user/fecthUser', "id" )
          this.$router.push("/mypage")
          // console.log("リダイレクトするはず")
        } else {
          this.$router.push("/login")
        }
      })
      
    },
    googleLogin () {
      const provider = new firebase.auth.GoogleAuthProvider()
      //これどっちがいいのか不明なので２つおいておく
      // firebase.auth().signInWithRedirect(provider)
      firebase.auth().signInWithPopup(provider)
          //       .then(url => {
          //   // アップロード完了処理 (ローカルメンバに保存したり)
          //   this.fileName = fileName
          //   this.imageUrl = url
          // })
      firebase.auth().onAuthStateChanged(user => {
      if (user) {
        // User is signed in.
        this.$store.dispatch('user/fecthUser', "id" )
        this.$router.push("/mypage")
        } else {
          this.$router.push("/login")
        }
      })
    }
  },
}

</script>