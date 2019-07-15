<template>
  <v-layout>
    <v-flex xs12 sm6 offset-sm3>
      <div style="padding-bottom:60px"><v-card>
      <v-toolbar fixed light height="">
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
    <div class="line__container">
      <div class="line__contents scroll" id ="chatarea">
      </div>
    </div>
    <v-container style="margin-top:20%">
      <v-flex xs11 style="width:100%;display:inline-block ">
      <v-text-field
        v-model="comment"
        :rules="nameRules"
        label=""
        required
      ></v-text-field>
      </v-flex>
      <v-flex xs1 style="display:inline-block" >
        <v-icon>send</v-icon>
      </v-flex>
    </v-container>
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
      sendMessage(){
        firebase.auth().onAuthStateChanged(user => {
            this.user = user ? user : {}
            //console.log(this.user.uid)
            const db = firebase.firestore()
            // ストレージオブジェクト作成
            var storageRef = firebase.storage().ref();
            //ファイル名が被らないように一意な名前を作成
            this.imageName = uuid();
            // ファイルのパスを設定
            var mountainsRef = storageRef.child(`images/${this.imageName}`);
            
        })
      },
      ichiran(){
        this.chat=[];
            const db = firebase.firestore()
             var docRef = db.collection("item").doc(this.itemId).collection("comment");
               docRef.get().then(querySnapshot => {
            querySnapshot.forEach(doc =>{
              // doc.data() is never undefined for query doc snapshots
              console.log(doc.id, " => ", doc.data());
              console.log(this.exChang);
              //ループでとる場合 * JSの場合
              // this.exChang.forEach(element => {
              //   console.log(element.comment);
                this.chat.push(doc.data().comment);
              // });
            });
          });
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
