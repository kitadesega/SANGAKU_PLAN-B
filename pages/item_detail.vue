<template>
  <v-layout>
    <v-flex v-if="loading" xs12 sm6 offset-sm3>
      <div style="padding-bottom:50px">
        <v-card>
            <v-toolbar fixed light height="">
              <nuxt-link :to="{path: '/'}">
                <v-btn icon >
                  <v-icon>arrow_back</v-icon>
                </v-btn>
              </nuxt-link>
              <v-toolbar-title class="notNewLine">{{item.item_name}}</v-toolbar-title>
              <v-spacer></v-spacer>
            </v-toolbar>
          </v-card>
      </div>
    <v-card>
      <v-spacer></v-spacer>
       <v-container grid-list-sm fluid>
           <v-flex xs12 sm2 md1>
             <nuxt-link :to="{path: '/select_user_mypage', 
             query: {
               userId: item.user_id,
               userPhoto: item.user_photo,
               userName: item.user_name,
              }}">
              <v-avatar
                size="36px"
              >
                <img
                  :src="item.user_photo"
                  alt="Avatar"
                >
              </v-avatar>&nbsp;&nbsp;<strong v-html="item.user_name"></strong>
             </nuxt-link>
            </v-flex>
          <v-layout row wrap>
            <v-flex>
            
              <v-card flat tile class="d-flex">

                <v-carousel
                  hide-delimiters
                  light
                  height="250px"
                >
                  <v-carousel-item
                    v-for="(item,i) in item.image_url"
                    :key="i"
                    :src="item"
                    height="200"
                    aspect-ratio="1"
                    class="grey lighten-2"
                  >
                  </v-carousel-item>
                </v-carousel>
              </v-card>
            </v-flex>
          </v-layout>
          <v-layout column fill-height >      
            <v-flex shrink xs1>
            <br>
            <div id="heading">商品名</div>
            </v-flex>
          </v-layout>
          <v-layout column fill-height >      
            <v-spacer></v-spacer>
              <v-flex shrink xs1>
              <div id="item_name">{{item.item_name}}</div>
              </v-flex>
          </v-layout>
          <v-layout column fill-height >      
              <v-flex shrink xs1>
              <br>
              <div id="heading">商品説明</div>
            </v-flex>
          </v-layout>
          <p id="syohin_text">{{item.item_text}}</p>
          <br>
          <v-layout column fill-height >      
              <v-flex shrink xs1>
              <br>
              <div id="heading">カテゴリー</div>
            </v-flex>
          </v-layout>
          <v-layout column fill-height >      
            <v-spacer></v-spacer>
              <v-flex shrink xs1>
              <div>{{item.category}}</div>
              </v-flex>
          </v-layout>
          <v-layout column fill-height >      
              <v-flex shrink xs1>
              <br>
              <div id="heading">国</div>
            </v-flex>
          </v-layout>
          <v-layout column fill-height >      
            <v-spacer></v-spacer>
              <v-flex shrink xs1>
              <div>{{item.county}}</div>
              </v-flex>
          </v-layout>
        </v-container>
    </v-card>
    <v-card height="60px" flat> 
      <v-footer height="60px" :fixed=true>

        <nuxt-link :to="{path: '/comment', query: {itemId: itemId }}">
          <v-btn round>コメント</v-btn>
        </nuxt-link>

        <v-btn @click="click" large color="yellow" type="submit">書き込み</v-btn>

        <nuxt-link :to="{path: '/request_edit', query: {itemId: itemId ,userId:item.user_id}}">
          <v-btn large dark color="red">申請を作成</v-btn>
        </nuxt-link>

      </v-footer>
    </v-card> 
    </v-flex>

    <v-progress-circular
    v-else
      indeterminate
      color="primary"
    ></v-progress-circular>
        <!-- dialogのやつ -->
    <v-layout row justify-center>
      <v-dialog v-model="dialog" persistent max-width="290">
        <v-card>
          <br><v-flex xs12>
          <v-textarea
            solo
            name="input-7-4"
            v-model="input"
            label="コメント欄"
            value=""
            height=150
          ></v-textarea>
          </v-flex>
          <div id ="kategori">
          <v-btn @click="dialog = false" @click.prevent="setUser(input)">送信</v-btn>
          <v-btn @click="dialog = false">中断</v-btn>
          </div>
        </v-card>
      </v-dialog>
    </v-layout>
  </v-layout>
</template>

<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import uuid from 'uuid'

export default {
layout: 'not_bottom',

  data() {
    return {
      user: {},  // ユーザー情報
      item: '',  // 商品一覧
      itemId : 'default ID',
      dialog: false,
      loading: false,
      chat:[]
    }
  },
  asyncData(context) {
    return {
      itemId: context.query['itemId']
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
        //userにログインしているユーザーのデータを入れる
        this.user = user ? user : {}
        //firestore設定
        const db = firebase.firestore()
        //itemコレクションを選択（コレクションについては各自調べてください）
        var docRef = db.collection("item").doc(this.itemId);
        
        docRef.get().then(doc => {
            if (doc.exists) {
                this.item = doc.data();
            } else {
                console.log("No such document!");
            }
            this.loading = true
        }).catch(function(error) {
            console.log("Error getting document:", error);
        });
    })
  },
    methods : {
      ...mapActions(['setUser']), 
      click() {
      this.dialog =　!this.dialog
    },
        setUser: function (data) {
        console.log(data);
        /****************   ここからしたで作った配列（連想配列）をFirebaseに登録（上のcreated）のとこ参照  ********************* */
        const db = firebase.firestore()
        db.collection("item").doc(this.itemId).collection("comment").doc().set({
                 comment: data
        })
      } 
    },
  
};


</script>

<style>
#heading{
background-color: lightgray;
color: #009999;
}
#item_name{
font-weight: 600;
}
#syohin_text{
   word-wrap: normal;
   display: inline-block;
   vertical-align: middle;
   word-break: break-all;
}
#kategori{
  text-align: center;
}
</style>
<style>
#syonin{
  text-align: center;
}
a{
  text-decoration-line: none;
}
</style>
