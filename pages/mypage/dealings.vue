<template>
      <v-container>

   <!-- ▼LINE風ここから -->
        <v-layout justify-center>
          <form action @submit.prevent="sendDealingsApproval" class="form">
          <v-btn :disabled="!checked" type="submit" large color="error">
              お土産を受け取りました
          </v-btn>
          </form>
          <br/>
        </v-layout>
        <v-layout justify-center>
          <div style="font-size:10px">※お互いが受け取ると取引完了です</div>
        </v-layout>
        <div class="line__container">
          
            <!-- タイトル -->
            <div class="line__title">
                チャットエリア
            </div>

            <!-- ▼会話エリア scrollを外すと高さ固定解除 -->
            <div class="line__contents scroll" id ="chatarea">
              <div v-for="(value,index) in text" :key="index">
              {{ value.name }}
              <div class="line__left">
                <div class="text">
                  <pre>{{value.message}}</pre>
                  <!-- <div v-html="$sanitize(value.message)"></div> -->
                </div>
              </div>          
            <!-- ここにチャットログが入る -->
            </div>

          </div>
            <!--　▲会話エリア ここまで -->

        </div>
    <!--　▲LINE風 ここまで -->
      <form action @submit.prevent="sendMessage" class="form">
          <v-flex xs12>
            <v-textarea
              outline
              name="input-7-4"
              label="何か送ってみましょう"
              value=""
              v-model="message"
            ></v-textarea>
          </v-flex>
          <v-layout justify-center>
            <v-btn type="submit" large color="error">メッセージを送る</v-btn>
          </v-layout>
      </form>
        
        
      </v-container>
      
</template>

<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import '@/assets/css/chat.css'
import sanitizeHTML from 'sanitize-html'
import Vue from 'vue'
Vue.prototype.$sanitize = sanitizeHTML

export default {
  
  fetch ({ store, route,redirect }) {
    if (!store.state.user.user) {
      if(route.name != "/login"){
        return redirect('/login')
      }else{
        return redirect('/mypage')
      }
    }
    
  },
  data() {
    return {
      user: {},  // ユーザー情報
      text: [],  // 取得したメッセージを入れる配列
      message: '',  // 入力したメッセージ
      title: '',
      photo: null,
      photo_url: null,
      dialog: false,
      imageName: "",
      imageUrl: "",
      imageFile: "",
      dealingsKey:"",
      dealingId:"",
      chatRoomId:"",
      checked:false,

    }
    //console.log(user);
  },
  asyncData(context) {
    return {
      chatRoomId: context.query['chatRoomId'],
      dealingsId: context.query['dealingId']
    }
  },
  
  created() {
    firebase.auth().onAuthStateChanged(user => {
        // User is signed in.
        //userにログインしているユーザーのデータを入れる
        this.user = user ? user : {}
        //firestore設定
        const db = firebase.firestore()
        
        //チャット情報
        var docRef = db.collection("chat").doc(this.chatRoomId).collection("messages").orderBy("created_at", "desc");
        
        //チャットデータ抽出
        docRef.onSnapshot(snapshot => {
            snapshot.docChanges().forEach(item => {
              this.text.push(item.doc.data());
            })
        })

        //取引のコレクション
         db.collection('users').doc(this.user.uid).collection('dealings').doc(this.dealingsId)
         .get().then(doc => {
            if (doc.exists) {
              this.dealingsKey = doc.data().dealings_id;
              console.log(doc.data().dealings_id);
            } else {
                console.log("No such document!");
            }
        })
        .then(_=> {
         db.collection("dealings").doc(this.dealingsKey)
         .get().then(doc => {
            if (doc.exists) {
              
            if(doc.data()[this.user.uid]){
              this.checked = false;
            }else{
              this.checked = true;
            }

            } else {
                console.log("No such document!");
            }
        })
        })

// console.log(this.dealingsKey);
    })
  },
  methods : {
    ...mapActions(['setUser']), 
    sendMessage(){
      // firebase.auth().onAuthStateChanged(user => {
        
        // this.user = user ? user : {}
        const db = firebase.firestore()
          var data = {
          id: this.user.uid,
          name: this.user.displayName,
          message: this.message,
          created_at:new Date(),
        };
        
        if(this.message != ""){
        var setDoc = db.collection("chat").doc(this.chatRoomId).collection("messages").doc().set(data);
        }
        this.message = "";

          
          
        
      // })
      
    },
    //受け取り処理
    sendDealingsApproval(){
      const db = firebase.firestore()
      var washingtonRef = db.collection("dealings").doc(this.dealingsKey);

      // Set the "capital" field of the city 'DC'
      washingtonRef.update({
          [this.user.uid]: true
      }).then(_=> {
        this.checked = false;
      });
    }
  },
  br2nl(){
      return str.replace(/\n/g, '<br>');
  }
  
};


</script>