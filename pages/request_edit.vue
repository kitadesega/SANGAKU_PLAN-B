<template>
  <v-card light>
      <div style="padding-bottom:50px">
        <v-card>
            <v-toolbar fixed light height="">
              <v-btn icon>
                <nuxt-link :to="{path: '/item_detail', query: {itemId: itemId }}">
                <v-icon>arrow_back</v-icon>
                </nuxt-link>
              </v-btn>
              <v-toolbar-title>申請作成</v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
          </v-card>
      </div>
    <v-card-actions>
      <v-spacer></v-spacer>
    </v-card-actions>
    <v-container grid-list-md text-xs-center>
      <h2 class="l-single-head">交換に出す候補を選択</h2>
      <p style="margin-top:-20px;padding-bottom:15px;font-size:14px">&emsp;最大3つまで選択できます</p>
    <form action @submit.prevent="sendRequest" class="form">
      <v-layout row wrap class="scroll">
        <v-flex
          xs4
          md3
          v-for="(value,index) in item" :key="index"
          style="margin-left:0px"
        > 

          <label :for="value.item_id">
            <v-card flat tile style="width:100%">
          <v-checkbox
          :id="value.item_id" 
          :value="value.item_id" 
          v-model="checkedItems"
          style="padding:0" 
          color="orange"
          v-on:change="selectCount"
          ></v-checkbox>
              <img
                :src= "value.image_url[0]"
                width="100%"
                height="100px"
                style = "object-fit: cover"
              >
              <p style="text-overflow: ellipsis;white-space: nowrap;overflow: hidden;">
                {{value.item_name}}
              </p>
            </v-card>
          </label>
        </v-flex>
      </v-layout>
      <p style="width:100%; margin:10px 0 13px 0; background-color:gray; color:white;line-height:200%">
        &emsp;メッセージを添えてみましょう
      </p>
      <v-flex xs12>
        <v-textarea
          solo
          name="input-7-4"
          v-model="message"
          label=""
          value=""
          height=150
        ></v-textarea>
      </v-flex>
    <v-btn type="submit" :disabled="!checked" large round color="yellow" >申請を送信</v-btn>
    </form>
    </v-container>
  </v-card>
</template>

<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import uuid from 'uuid'

export default {
    //ボトムナビ無し
  layout: 'not_bottom',
      fetch ({ store, route,redirect }) {
    if (!store.state.user.user) {
      //console.log("リダイレクトなんだよなぁ")
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
      item: [],  // 商品一覧
      itemDetail :[],
      message: '',  // 入力したメッセージ
      dialog: false,
      checkedItems: [],
      itemId : 'default ID',
      userId : 'default ID',
      checked:true,
      docRefId:'',
    }
  },
  asyncData(context) {
    return {
      itemId: context.query['itemId'],
      userId: context.query['userId']
    }
  },

  created() {
    firebase.auth().onAuthStateChanged(user => {
      //userにログインしているユーザーのデータを入れる
      this.user = user ? user : {}
      //firestore設定
      const db = firebase.firestore()
      //itemコレクションを選択（コレクションについては各自調べてください）
      var docRef = db.collection("users/"+this.user.uid+"/item").where("display_flg", "==", true);
      // var query = docRef.orderBy("created_at", "asc");

      
      docRef.onSnapshot(snapshot => {
        snapshot.docChanges().forEach(item => {
          this.item.push(item.doc.data());
          console.log(item.doc.data());
        })
      })

      var docRef2 = db.collection("item").doc(this.itemId);
      
      docRef2.get().then(doc => {
          if (doc.exists) {
              this.itemDetail = doc.data();
          } else {
              console.log("No such document!");
          }
      }).catch(function(error) {
          console.log("Error getting document:", error);
      });
     })



  },
    methods : {
      ...mapActions(['setUser']), 
      selectCount(id){
        console.log(this.checkedItems)
        if(this.checkedItems.length == 0){
          this.checked = false;
        }else if(this.checkedItems.length <= 3){
          this.checked = true;
        }else{
          this.checked = false;
        }
    },
      sendRequest(){
            const db = firebase.firestore()

          //送る側の申請データ
            const data2 = {
            user_id: this.user.uid,
            user_name: this.user.displayName,
            user_photo: this.user.photoURL,
            target_item_id: this.itemId,
            target_item_image :this.itemDetail.image_url,
            item1_id: this.checkedItems[0] ? this.checkedItems[0] : '',
            item2_id: this.checkedItems[1] ? this.checkedItems[1] : '',
            item3_id: this.checkedItems[2] ? this.checkedItems[2] : '',
            text: this.message,
            created_at:new Date(),
          };
          //自分が送った申請
          db.collection('users').doc(this.user.uid).collection('sendRequest').add(data2)
          .then(docRef => {
            this.docRefId = docRef.id
          }).then(_=>{
            //送られた側に入れるデータ
            const data4 = {
            user_id: this.user.uid,
            user_name: this.user.displayName,
            user_photo: this.user.photoURL,
            target_item_id: this.itemId,
            item1_id: this.checkedItems[0] ? this.checkedItems[0] : '',
            item2_id: this.checkedItems[1] ? this.checkedItems[1] : '',
            item3_id: this.checkedItems[2] ? this.checkedItems[2] : '',
            text: this.message,
            created_at:new Date(),
          };
            const data5 = {
              image_url:this.itemDetail.image_url,
              message:this.user.displayName + "さんから申請が届きました。",
              type:"request",
              link_id:this.docRefId,
              read_flag:false,
              created_at:new Date(),
            }
          db.collection('users').doc(this.userId).collection('notice').doc(this.docRefId).set(data5)
          db.collection('users').doc(this.userId).collection('request').doc(this.docRefId).set(data4)
            .then(_ => {
              this.$router.push("/")
              this.user = '';
              this.itemId = '';
              this.message = '';
            });
          })

          //どれか１つでも初期化しないとアカウント切り替え時にバグる




      },
    },
  
};
</script>
<style>

  .scroll {
  height: 400px;
  overflow-y: scroll;
  }
  .l-single-head {
  padding: 24px 8px;
  font-size: 18px;
  line-height: 1.5;
  text-align: center;
}
.notNewLine{
  text-align:center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>