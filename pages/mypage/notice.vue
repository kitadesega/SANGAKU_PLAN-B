<template>
  <div>

        <div>
          <v-toolbar>
            <v-toolbar-title>お知らせ</v-toolbar-title>
            <v-spacer></v-spacer>
          </v-toolbar>
        </div>
         <v-list three-line>
            <div v-for="(item, index) in item" :key="index" >
              <div :v-if="item.type == 'request'">
                <nuxt-link :to="{path: '/mypage/request_detail', query: {requestId: item.link_id }}">
                      <v-list-tile
                    :key="item.link_id"
                  >
                    <v-list-tile>
                      <img :src="item.image_url[0]"
                      width="50px"
                      height="50px">
                    </v-list-tile>

                    <v-list-tile-content>
                      
                      <v-list-tile-sub-title v-html="item.message"></v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                </nuxt-link>
              </div>
              <div :v-else-if="item.type == 'dealings'">
              <nuxt-link :to="{path: '/mypage/dealings', query: {
                chatRoomId: item.chatroom_id,
                dealingId : item.dealings_id,
                dealingsKey:item.dealings_key}}">
                      <v-list-tile
                    :key="item.dealings_key"
                  >
                    <v-list-tile>
                      <img src="item.image_url"
                      width="50px"
                      height="50px">
                    </v-list-tile>

                    <v-list-tile-content>
                      
                      <v-list-tile-sub-title v-html="item.message"></v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                </nuxt-link>
              </div>
            </div>
          </v-list>
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
            // console.log(item.doc.data())
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