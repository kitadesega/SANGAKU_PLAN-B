<template>
  <v-app>

    <!-- <v-toolbar :fixed="fixed" app >
      <v-spacer />
        <v-text-field
        hide-details
        prepend-icon="search"
        single-line
        placeholder="何をお探しですか？"
         />
        <v-spacer />
    </v-toolbar> -->
    
    <v-content>
        <nuxt />
    </v-content>

 <v-card height="60px" flat> 
   <v-footer>
 <v-bottom-nav :value="true" :active.sync="bottomNav"
 :fixed=true >
      <v-btn color="teal" flat value="recent" to = "/" >
        <span>ホーム</span>
        <v-icon>home</v-icon>
      </v-btn>

      <v-btn
        color="teal"
        flat
        value="unko"
        to = "/mypage/notice"
      >
        <span>お知らせ</span>
        <v-badge color="red">
      <template v-slot:badge>
        <span>{{noticeCount}}</span>
      </template>
        <v-icon>notifications_none</v-icon>
           </v-badge>
      </v-btn>

      <v-btn
        color="teal"
        flat
        value="favorites"
        to="/item_create"
      >
        <span>出品</span>
        <v-icon>photo_camera</v-icon>
      </v-btn>

      <v-btn
        color="teal"
        flat
        value="unko"
        to = "/mypage/"
      >
        <span>マイページ</span>
        <v-icon>accessibility</v-icon>
      </v-btn>
    </v-bottom-nav>
    </v-footer>
    </v-card>

  </v-app>
  
</template>

<script>
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
export default {
// データをこのコンポーネントにセットする必要がないため fetch() を使う

 data() {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      absolute:false,
      value:true,
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: '産学連携',
      bottomNav: "",
      color: '#FFF',
      noticeCount:'',
      count:[]
    }
  },
    created() {
    firebase.auth().onAuthStateChanged(user => {
      if (user) {
        // User is signed in.
        this.user = user ? user : {}
        const db = firebase.firestore()
        //itemコレクションを選択（コレクションについては各自調べてください）
        var docRef = db.collection('users').doc(this.user.uid).collection('notice');
        //データ取得の条件を指定して取得

         //通知数獲得
        docRef.onSnapshot(snapshot => {
             this.count = [];
            this.noticeCount = 0;
            snapshot.docChanges().forEach(item => {

              if(item.doc.data().read_flag == false){
                console.log(item.doc.data());
              this.count.push(item.doc.data());
              this.noticeCount = this.count.length;
              }else{
              this.noticeCount = 0;
              }
            });
        })
        
      } else {
        // No user is signed in.
        this.$router.push("/login")
      }
    })
  }
}
</script>
