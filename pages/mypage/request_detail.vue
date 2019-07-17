<template>

  <v-container grid-list-md text-xs-center>
    <div style="padding-bottom:50px">
      <v-card>
        <v-toolbar fixed light height="">
          <v-btn icon>
            <nuxt-link :to="{path: backLink}">
            <v-icon>arrow_back</v-icon>
            </nuxt-link>
          </v-btn>
          <v-toolbar-title>受信した申請</v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
      </v-card>
    </div>
  <form action @submit.prevent="sendRequestApproval" class="form">
    <maincard />
    <div class="item">
        <p> 出品した商品 </p>
        
        <img
        :src= "targetItem.image_url"
        width="200px"
        height="100px"
        style = "object-fit: cover"       
        >
    </div>
    <p>{{targetItem.item_name}}</p>
    <v-layout row wrap >
      
      <v-flex
        xs4
        md3
      >
        <v-card flat tile style="width:100%">
            <nuxt-link :to="{path: '/item_detail', query: {itemId: item1.item_id }}">
          <img
            :src= "item1.image_url"
            width="100%"
            height="100px"
            style = "object-fit: cover"
          >
          <p class="notNewLine">{{item1.item_name}}</p>
            </nuxt-link>
            <v-layout justify-center>
              <input type="radio" :id="item1.item_id" :value="item1.item_id" v-model="picked" style="display:none;">
              <v-btn :color="btnColor1" v-on:click="itemSelect(item1,'item1')">これならOK</v-btn>
            </v-layout>
        </v-card>
      </v-flex>

      <v-flex
        xs4
        md3
      >
        <v-card flat tile style="width:100%" v-show="item2.item_id">
            <nuxt-link :to="{path: '/item_detail', query: {itemId: item2.item_id }}">
          <img
            :src= "item2.image_url"
            width="100%"
            height="100px"
            style = "object-fit: cover"
          >
          <p class="notNewLine">{{item2.item_name}}</p>
            </nuxt-link>
            <v-layout justify-center>
              <input type="radio" :id="item2.item_id" :value="item2.item_id" v-model="picked" style="display:none;">
              <v-btn :color="btnColor2" v-on:click="itemSelect(item2,'item2')">これならOK</v-btn>
            </v-layout>
        </v-card>
         
      </v-flex>
      <v-flex
        xs4
        md3
      >
        <v-card flat tile style="width:100%" v-show="item3.item_id">
          <nuxt-link :to="{path: '/item_detail', query: {itemId: item3.item_id }}">
          <img
            :src= "item3.image_url"
            width="100%"
            height="100px"
            style = "object-fit: cover"
          >
          <p class="notNewLine">{{item3.item_name}}</p>
          </nuxt-link>
          <v-layout justify-center>
            <input type="radio" :id="item3.item_id" :value="item3.item_id" v-model="picked" style="display:none;">
            <v-btn :color="btnColor3" v-on:click="itemSelect(item3,'item3')">これならOK</v-btn>
          </v-layout>
        </v-card>
      </v-flex>
    </v-layout>
   <p>{{requestData.text}}</p>
    <!-- <span>Picked: {{ picked }}</span> -->
    <v-btn class="align-center" type="submit" :disabled="!checked" large round color="yellow" >申請を許可</v-btn>
  </form>
  </v-container>
</template>
<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import uuid from 'uuid'
export default {
  //ボトムナビ無しver
  layout: 'not_bottom',

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
      requestData: {},  // 商品一覧
      items: [],
      item1:{},
      item2:{},
      item3:{},
      targetItem:{},
      requestId : 'default ID',
      dialog: false,
      picked:'',
      checked:false,
      btnColor1:"",
      btnColor2:"",
      btnColor3:"",
      backLink : ""
    }
  },
  asyncData(context) {
    return {
      requestId: context.query['requestId'],
      backLink: context.query['backLink']
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
      // User is signed in.
      //userにログインしているユーザーのデータを入れる
      this.user = user ? user : {}
      //firestore設定
      const db = firebase.firestore()
      //itemコレクションを選択（コレクションについては各自調べてください）
      var docRef = db.collection("users").doc(this.user.uid).collection("request").doc(this.requestId);
      docRef.get().then(doc => {
          if (doc.exists) {
            this.requestData = doc.data()
          } else {
            console.log("No such document!");
          }
      }).then(_=>{
        const item1_id = this.requestData.item1_id;
        const item2_id = this.requestData.item2_id;
        const item3_id = this.requestData.item3_id;
        const itemDocRef = db.collection("item");

        
        itemDocRef.doc(this.requestData.target_item_id).get().then(doc => {
          let data = {
            'item_id': doc.id ? doc.id: false,
            'item_name': doc.data().item_name ? doc.data().item_name : '',
            'image_url': doc.data().image_url ? doc.data().image_url[0] : '',
          }
          this.targetItem = data;
        })

        itemDocRef.doc(item1_id).get().then(doc => {
          let data = {
            'item_id': doc.id ? doc.id: false,
            'item_name': doc.data().item_name ? doc.data().item_name : '',
            'image_url': doc.data().image_url ? doc.data().image_url[0] : '',
          }
          this.item1 = data;
        })

        itemDocRef.doc(item2_id).get().then(doc => {
          let data = {
            'item_id': doc.id ? doc.id: false,
            'item_name': doc.data().item_name ? doc.data().item_name : '',
            'image_url': doc.data().image_url ? doc.data().image_url[0] : '',
          }
          this.item2= data;
        })

        itemDocRef.doc(item3_id).get().then(doc => {
          let data = {
            'item_id': doc.id ? doc.id: false,
            'item_name': doc.data().item_name ? doc.data().item_name : '',
            'image_url': doc.data().image_url ? doc.data().image_url[0] : '',
          }
          this.item3 = data;
        })
      });     
    });
    
  },
    methods : {
      ...mapActions(['setUser']), 

      //ラジオボタン関連の処理
      itemSelect(checkItem,itemPosition){
        this.picked = checkItem
        this.checked = true
        console.log(this.picked);
        this.btnColor1 = itemPosition == "item1" ? "success" : "";
        this.btnColor2 = itemPosition == "item2" ? "success" : "";
        this.btnColor3 = itemPosition == "item3" ? "success" : "";
      },
      sendRequestApproval(){
        firebase.auth().onAuthStateChanged(user => {
            this.user = user ? user : {}
            const db = firebase.firestore()
            const chatId = uuid();

            let dealings_id = uuid();
            const data = {
              //相手の
              target_user_id: this.requestData.user_id,
              target_user_name: this.requestData.user_name,
              target_user_photo: this.requestData.user_photo,
              target_item_id: this.picked.item_id,
              target_item_name: this.picked.item_name,
              target_item_image: this.picked.image_url,
              //自分の
              user_id: this.user.uid,
              item_id : this.targetItem.item_id,
              item_name: this.targetItem.item_name,
              item_image:this.targetItem.image_url,
              //チャットのキー
              chatroom_id : chatId,
              //取引キー
              dealings_id : dealings_id,
              created_at:new Date(),
          };

            const data2 = {
              //相手の
              target_user_id: this.user.uid,
              target_user_name: this.user.displayName,
              target_user_photo: this.user.photoURL,
              target_item_id: this.targetItem.item_id,
              target_item_name: this.targetItem.item_name,
              target_item_image: this.targetItem.image_url,
              //相手から見た自分の
              user_id: this.requestData.user_id,
              item_id : this.picked.item_id,
              item_name: this.picked.item_name,
              item_image: this.picked.image_url,
              //チャットのキー
              chatroom_id : chatId,
              dealings_id : dealings_id,
              created_at:new Date(),
          };

          const data3 = {
            [this.user.uid] : false,
            [this.requestData.user_id] : false,
          }



          //受け取り判定に使う取引コレクション
          db.collection('dealings').doc(dealings_id).set(data3)

          //取引が開始になるので送られたリクエストを削除
          db.collection("users").doc(this.user.uid).collection("request").doc(this.requestId).delete()

          //相手側の送信したリクエストを削除
          db.collection("users").doc(this.user.uid).collection("sendRequest").doc(this.requestId).delete()

          //相手側に取引データを入れる
          db.collection('users').doc(this.requestData.user_id).collection('dealings').doc().set(data2)

          //自分に取引データを入れるここのid欲しい、通知に使う
          db.collection('users').doc(this.user.uid).collection('dealings').add(data)
          .then(docRef => {
             const data5 = {
              //取引ID
              dealings_id:docRef.id,
              //チャットのキー
              chatroom_id : chatId,
              //取引キー
              dealings_key : dealings_id,
              image_url:this.picked.image_url,
              message:this.requestData.user_name + "さんとの取引が開始しました",
              read_flag:false,
              type:"dealings",
              created_at:new Date(),
          };

          //通知機能に使う奴
          db.collection('users').doc(this.requestData.user_id).collection('notice').doc().set(data5)


          this.user = '';
          this.picked = '';
          this.requestData = '';
          dealings_id = '';
            this.$router.push("/")
          });
        })
      },
    },
  
};
</script>

<style scoped>
.item {
  padding: 0;
  color: #2c2c2f;
  /* background: red; 背景色 */
}
.item p {
  font-size: 20px;
  margin: 0;
  padding: 0;
}


/*改行させるかよ*/
.notNewLine{
  text-align:center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>

