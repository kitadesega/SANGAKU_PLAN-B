<template>
  <v-layout>
<v-flex xs12 sm6 offset-sm3>
    <v-toolbar :fixed="true" app >
      <v-spacer />
        <v-text-field
        hide-details
        prepend-icon="search"
        single-line
        placeholder="何をお探しですか？"
        v-model="searchWood"
         />
        <v-spacer />
    </v-toolbar>

    
      <v-card-actions>
        <v-spacer></v-spacer>
      </v-card-actions>
      <v-container grid-list-md text-xs-center v-if="loading">
        
        <v-layout row wrap >
          <v-flex
          xs4
          v-for="(value,index) in filteredUsers" :key="index"
          style="margin-bottom:0px"
          >
              <v-card light tile style="overflow: hidden"  v-bind:class="{ triangle: !value.display_flg }">
              <nuxt-link :to="{path: '/item_detail', query: {itemId: value.itemId }}">
                <img
                  :src= "value.image_url[0]"
                  style = "object-fit: cover"
                  width = "100%"
                  height = "110px"
                >
                <p style="background-color:darkgrey;">{{value.country}}</p>
                <p style="text-overflow: ellipsis;white-space: nowrap;overflow: hidden;">
                  <span v-if="value.display_flg === false" style="color:red;">
                    </span>&nbsp;{{value.item_name}}</p>
                </nuxt-link>
                </v-card>
          </v-flex>
        </v-layout>
        
      </v-container>
      <v-progress-circular
      v-else
      :size="70"
      :width="7"
      color="purple"
      indeterminate
      style="position:absolute;margin-left:40%;margin-top:100px"
    ></v-progress-circular>
    
    
</v-flex>

  </v-layout>
</template>

<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'

export default {

    data() {
    return {
      searchWood: "",
      user: {},  // ユーザー情報
      item: [],  // 商品一覧
      dialog: false,
      loading: false
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
      // User is signed in.
      //userにログインしているユーザーのデータを入れる
      this.user = user ? user : {}
      //firestore設定
      console.log(this.$store.state.user.user)
      const db = firebase.firestore()
      //itemコレクションを選択（コレクションについては各自調べてください）
      var docRef = db.collection("item").orderBy("created_at", "desc");
      //データ取得の条件を指定して取得

        //データ取得
      docRef.onSnapshot(snapshot => {
          snapshot.docChanges().forEach(item => {
              let data = {
              'itemId': item.doc.id,
              'item_name': item.doc.data().item_name,
              'image_url': item.doc.data().image_url,
              'country': item.doc.data().country,
              'display_flg': item.doc.data().display_flg,
            }
            this.item.push(data);
            this.loading = true
          });
      })
    })
  },
    methods : {
      ...mapActions(['setUser']), 

      searchToolbar(){
        if (searchText != '') {
        }
      }

    },

    computed : {
      filteredUsers: function() {
        var users = [];
        for(var i in this.item) {
          var user = this.item[i];
          if(user.item_name.indexOf(this.searchWood) !== -1) {
            users.push(user);
          } 
        }
        return users;
      }    
    }
  
};
</script>
<style scoped>
a{
  text-decoration: none !important;
  color:black;
}
p{
  font-size:12px;
  margin-bottom:0px !important;
}
/*改行させるかよ*/
.notNewLine{
  width:50px;
  text-align:center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 商品に見立てた枠 */
.triangle {
    /* background: #fafafa;
    width: 250px;
    height: 250px;
    border: #eee 1px solid;
    margin: 0 auto; */
    position: relative;
}

/* ラベル部分 左上に表示 */
.triangle::before {
    content: "";
    top: 0;
    left: 0;
    border-bottom: 3.5em solid transparent;
    border-left: 3.5em solid #ea352d; /* ラベルの色はここで変更 */
    position: absolute;
    z-index: 100;
}
.triangle::after {
    content: "取引済";
    font-size: 0.8em; 
    display: block;
    top: 10px;
    transform: rotate(-45deg);
    color: #fff; /* 文字色はここで変更 */
    left: 0;
    position: absolute;
    z-index: 101;
}
</style>