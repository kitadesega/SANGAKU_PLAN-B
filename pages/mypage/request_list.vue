<template>

  <div>
      <div style="padding-bottom:50px">
        <v-card>
            <v-toolbar fixed light height="">
              <v-btn icon>
                <nuxt-link :to="{path: '/mypage'}">
                <v-icon>arrow_back</v-icon>
                </nuxt-link>
              </v-btn>
              <v-toolbar-title>お土産取引</v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
          </v-card>
      </div>
    <v-tabs
      centered
      dark
      icons-and-text
      v-model="tab"
       fixed-tabs
    >
      <v-tabs-slider color="yellow"></v-tabs-slider>
      <v-tab href="#tab-1">
        受信した申請
      </v-tab>
      <v-tab href="#tab-2">
        取引中
      </v-tab>
      <v-tab href="#tab-3">
        送った申請
      </v-tab>

    </v-tabs>
    <v-tabs-items v-model="tab">
      <!-- 送られてきた申請 -->
      <v-tab-item value="tab-1">
          <v-list three-line>
            <div v-for="(item, index) in item" :key="index" >
              <nuxt-link :to="{path: '/mypage/request_detail', query: 
              {requestId: item.request_id,
              backLink: backLink}}">
                <v-list-tile
                  :key="item.user_name"
                  user_photo
                >
                  <v-list-tile-avatar>
                    <img :src="item.user_photo">
                  </v-list-tile-avatar>

                  <v-list-tile-content>
                    <v-list-tile-title v-html="item.user_name"></v-list-tile-title>
                    <v-list-tile-sub-title v-html="item.text"></v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
              </nuxt-link>
            </div>
          </v-list>
      </v-tab-item>
      <!-- 取引中 -->
      <v-tab-item value="tab-2">
          <v-list three-line>
            <div v-for="(item, index) in dealings" :key="index" >
              <nuxt-link :to="{path: '/mypage/dealings', query: {chatRoomId: item.chatroom_id,
              dealingId : item.dealings_id, dealingsKey:item.dealings_key,backLink:backLink}}">
                <v-list-tile
                  :key="item.item_id"
                  target_user_photo
                >
                  <v-list-tile>
                    <img :src="item.item_image"
                    width="100px"
                    height="70px"
                    style = "object-fit: cover">
                  </v-list-tile>

                  <v-list-tile-content>
                    <v-list-tile-title v-html="item.target_user_name"></v-list-tile-title>
                    <v-list-tile-sub-title v-html="item.item_name"></v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
              </nuxt-link>
            </div>
          </v-list>
      </v-tab-item>
      <!-- 送信した申請 -->
        <v-tab-item value="tab-3">
          <v-list three-line>
            <div v-for="(item, index) in myRequest" :key="index" >
              <!-- <nuxt-link :to="{path: '/mypage/dealings', query: {dealingsId: item.chatroom_id }}"> -->
                <v-list-tile
                  :key="item.item_id"
                  target_user_photo
                >
                  <v-list-tile>
                    <img :src="item.target_item_image"
                    width="100px"
                    height="70px"
                    style = "object-fit: cover"
                    >
                  </v-list-tile>

                  <v-list-tile-content>
                    <v-list-tile-title v-html="item.target_user_name"></v-list-tile-title>
                    <v-list-tile-sub-title v-html="item.text"></v-list-tile-sub-title>
                  </v-list-tile-content>
                </v-list-tile>
              <!-- </nuxt-link> -->
            </div>
          </v-list>
      </v-tab-item>
    </v-tabs-items>
  </div>
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
      dealings:[],
      myRequest:[],
      tab: null,
      backLink: '/mypage/request_list'
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
      var docItemRef = db.collection('users').doc(this.user.uid).collection('request');
      //取引リスト
      var docDealingsRef = db.collection('users').doc(this.user.uid).collection('dealings');
      //送ったリクエスト一覧
      var docMyRequestRef = db.collection('users').doc(this.user.uid).collection('sendRequest');

      // 来た申請一覧
      docItemRef.onSnapshot(snapshot => {
          snapshot.docChanges().forEach(item => {
            let data = {
            'request_id': item.doc.id,
            'user_name': item.doc.data().user_name,
            'user_photo': item.doc.data().user_photo,
            'text': item.doc.data().text
          }
            this.item.push(data);
          })
      })

      // 取引中一覧
      docDealingsRef.onSnapshot(snapshot => {
          snapshot.docChanges().forEach(item => {
            let data = {
            'dealings_id' : item.doc.id,
            'dealings_key': item.doc.data().dealings_id,
            'item_id': item.doc.data().item_id,
            'item_image': item.doc.data().item_image,
            'item_name': item.doc.data().item_name,
            'target_user_name': item.doc.data().target_user_name,
            'chatroom_id': item.doc.data().chatroom_id,
          }
            this.dealings.push(data);
          })
      })

      //自分が送った申請一覧
      docMyRequestRef.onSnapshot(snapshot => {
          snapshot.docChanges().forEach(item => {
            this.myRequest.push(item.doc.data());
          })
      })
    })
  },
    methods : {
      ...mapActions(['setUser']), 
    },
  
};

</script>
<style>
a{
  text-decoration: none !important;
}

/*改行させるかよ*/
.notNewLine{
  text-align:center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>