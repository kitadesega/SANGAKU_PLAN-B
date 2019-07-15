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

  </v-app>
  
</template>

<script>
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import bottomNav from '~/components/bottomNav'
export default {
// データをこのコンポーネントにセットする必要がないため fetch() を使う
  components: {
    bottomNav,

  },
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
      noticeCount:0,
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
        var docRef = db.collection('users').doc(this.user.uid).collection('notice').where("read_flag","==",false)
        //データ取得の条件を指定して取得

         //通知数獲得
        // docRef.onSnapshot(snapshot => {``
        //     snapshot.forEach(item => {
        //       console.log(item.doc.data());
        //       // if(item.doc.data().read_flag == false){
        //         console.log(item.doc.data());
        //       // this.count.push(item.doc.data());
        //       console.log("変化あっただろ")
        //       tmp = tmp +1;
        //       this.noticeCount = tmp;
        //       // }
             
        //     });
        // })
db.collection('users').doc(this.user.uid).collection('notice').where("read_flag","==",false)
    .onSnapshot(querySnapshot=> {
        this.noticeCount = 0
        querySnapshot.forEach(doc => {
            console.log(doc.data())
            this.noticeCount = this.noticeCount + 1
        });
    });
        
        
      } else {
        this.noticeCount = 0
        this.$router.push("/login")
      }
    })
  }
}
</script>
