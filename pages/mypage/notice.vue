<template>
  <div>
    <div v-for="(value, index) in item" :key="index" >
      <p>{{value.message}}</p>
      <nuxt-link :to="{path: '/mypage/request_detail', query: {requestId: value.link_id }}">
      {{value.link_id}}
      </nuxt-link>
    </div>
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
      item:[]
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
        //userにログインしているユーザーのデータを入れる
      this.user = user ? user : {}
      //firestore設定
      const db = firebase.firestore()
      //itemコレクションを選択（コレクションについては各自調べてください）
      var docRef = db.collection('users').doc(this.user.uid).collection('notice')
      //データ取得の条件を指定して取得

        //データ取得
      docRef.onSnapshot(snapshot => {
          snapshot.docChanges().forEach(item => {
            this.item.push(item.doc.data());
            console.log(item.doc.data())
          });
      })

      db.collection('users').doc(this.user.uid).collection('notice').get().then(querySnapshot => {
        querySnapshot.forEach(doc => {
        var cityRef = db.collection('users').doc(this.user.uid).collection('notice').doc(doc.id);
        return cityRef.update({
            read_flag: true
        });
    });
});
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