<template>
  <div>
    <v-card>
      <v-avatar
    size="56px" >
        <img
          :src="userPhoto"
        >
        </v-avatar><a class="notNewLine">
        {{userName}}
      </a>
    </v-card>
    <v-flex xs12 sm6 offset-sm3>

    <v-card light width=100%>
      <v-card-actions>
        <v-spacer></v-spacer>
      </v-card-actions>
      <v-container grid-list-md text-xs-center>
        
        <v-layout row wrap >
          <v-flex
            xs4
            md3
            v-for="(value,index) in filteredUsers" :key="index"
            style="margin-left:0px"
          >
            <v-card flat tile style="width:100%">
              <nuxt-link :to="{path: '/item_detail', query: {itemId: value.itemId }}">
              <img
                :src= "value.image_url[0]"
                style = "object-fit: cover"
                width="100%"
                height="100px"
                
              >
              <p style="text-align:center">{{value.title}}</p>
              </nuxt-link>
            </v-card>
          </v-flex>
        </v-layout>
        
      </v-container>

    </v-card>
    
</v-flex>
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
      item:[],
      userId:'',
      searchWood: "",
      userPhoto:'',
      userName:''

    }
  },
  asyncData(context) {
    return {
      userId: context.query['userId'],
      userPhoto: context.query['userPhoto'],
      userName: context.query['userName'],
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
      var docRef = db.collection("item").where("user_id", "==", this.userId);
      //データ取得の条件を指定して取得

      //データ取得
      docRef.onSnapshot(snapshot => {
          snapshot.docChanges().forEach(item => {
              let data = {
              'itemId': item.doc.id,
              'title': item.doc.data().item_name,
              'image_url': item.doc.data().image_url
            }
            this.item.push(data);
          });
      })
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
  computed : {
      filteredUsers: function() {
        var users = [];
        for(var i in this.item) {
          var user = this.item[i];
          if(user.title.indexOf(this.searchWood) !== -1) {
            users.push(user);
          } 
        }
        return users;
      }    
    }
  
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