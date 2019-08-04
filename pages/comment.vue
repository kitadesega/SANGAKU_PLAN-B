<template>
  <v-layout>
    <v-flex xs12 sm6 offset-sm3>
      <div style="padding-bottom:60px"><v-card>
      <v-toolbar fixed light>
        <v-btn icon>
          <nuxt-link :to="{path: '/item_detail', query: {itemId: itemId }}">
          <v-icon>arrow_back</v-icon>
          </nuxt-link>
        </v-btn>
        <v-toolbar-title>コメント</v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
      </v-card>
    </div>
    <div class="line__container" style="height:auto">
      <div class="line__contents scroll" id ="chatarea" style="height:auto">
        <v-list three-line>
            <v-list-tile
                 v-for="(value, index) in chat" :key="index"
                avatar
                style="height:auto;"
            >
              <v-list-tile-avatar>
                <img :src="value.userPhoto">
              </v-list-tile-avatar>

              <v-list-tile-content style="height:auto;">
                <v-list-tile-sub-title style="margin:0 0 10px 0;">{{value.userName}}</v-list-tile-sub-title>
                <v-list-tile-sub-title class="text--primary subheading" style="margin:0 0 5px 0;">{{value.chatMessage}}</v-list-tile-sub-title>
                <v-list-tile-sub-title style="height:auto">
                  {{value.createdAt.toDate().toLocaleString()}}
                </v-list-tile-sub-title>
              </v-list-tile-content>

            </v-list-tile>
        </v-list>
      </div>
    </div>
    <v-card height="60px" flat > 
      <v-footer height="80px" :fixed=true>
          <v-flex xs10 style="padding:0 0 0 20px;width:50%;display:inline-block ">
          <v-text-field
            v-model="input"
            label=""
            required
          ></v-text-field>
          </v-flex>
          <v-flex xs2 style="display:inline-block" >
              <v-icon @click.prevent="sendChat(input)">send</v-icon>
          </v-flex>
      </v-footer>
    </v-card> 
  </v-flex>
</v-layout>
  
</template>

<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import uuid from 'uuid'
import '@/assets/css/chat.css'
  export default {
    layout: 'not_bottom',
    data() {
    return {
      user: {},  // ユーザー情報
      item: '',  // 商品一覧
      input: '',  // 入力したメッセージ
      itemId : 'default ID',
      dialog: false,
      exChang:[],
      chat:[]
    }
    //console.log(user);
  },
    asyncData(context) {
    return {
      itemId: context.query['itemId']
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
        // User is signed in.
        //userにログインしているユーザーのデータを入れる
        this.user = user ? user : {}
       this.ichiran();  
    })
    
  },
    methods : {
      ...mapActions(['setUser']),
      sendChat: function (data) {
        this.input = "";
        if(data){
          firebase.auth().onAuthStateChanged(user => {
            this.user = user ? user : {}
            console.log(data);
            const db = firebase.firestore()
            db.collection("item").doc(this.itemId).collection("chat").doc().set({
              user_id: user.uid,
              user_photo: user.photoURL,
              user_name: user.displayName,
              chat_message: data,
              display_flg: true,
              created_at: new Date(),
            })
          })
        } 
      }, 
      ichiran(){
        this.chat=[];
        firebase.auth().onAuthStateChanged(user => {
          // User is signed in.
          //userにログインしているユーザーのデータを入れる
          this.user = user ? user : {}
          //firestore設定
          const db = firebase.firestore()
          //itemコレクションを選択
          var docRef = db.collection("item").doc(this.itemId)
            .collection("chat").orderBy("created_at", "asc");
          //データ取得の条件を指定して取得

            //データ取得
          docRef.onSnapshot(snapshot => {
              snapshot.docChanges().forEach(chat => {
                  let data = {
                  'userId': chat.doc.user_id,
                  'userPhoto': chat.doc.data().user_photo,
                  'userName': chat.doc.data().user_name,
                  'chatMessage': chat.doc.data().chat_message,
                  'displayFlg': chat.doc.data().display_flg,
                  'createdAt': chat.doc.data().created_at,
                }
                console.log(data)
                this.chat.push(data);
              });
          })
        })
      }
    },
  
};

</script>

<style>
#kategori{
  text-align: center;
}
</style>
<style>
#comments{
  text-align: left;
}

</style>
