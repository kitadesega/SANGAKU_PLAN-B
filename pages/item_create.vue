<template>
  <v-layout>

    <v-flex >
      <form action @submit.prevent="sendMessage" class="form">
      <p style="width:100%; margin:10px 0 13px 0; background-color:gray; color:white;line-height:200%">
        &emsp;商品名を入力してください
      </p>
      <v-flex xs12 sm6 md3>
        <v-text-field
          label=""
          solo
          v-model="title"
        ></v-text-field>
      </v-flex>

      <p style="width:100%; margin:10px 0 13px 0; background-color:gray; color:white;line-height:200%">
        &emsp;画像を追加してください
      </p>
      <img
        width=33%
        v-show="flag"
        class="preview-item-file1"
        :src="uploadedImage[0]"
      /><img
        width=33%
        v-show="flag"
        class="preview-item-file2"
        :src="uploadedImage[1]"
      /><img
        width=33%
        v-show="flag"
        class="preview-item-file3"
        :src="uploadedImage[2]"
      />
        <v-layout justify-center>
          <v-btn
            @click="pickFile"
            v-model="imageName"
            fab
          >
            <v-icon>photo_camera</v-icon>
          </v-btn>
        </v-layout>
      <input
        type="file"
        style="display: none"
        ref="image"
        accept="image/*"
        @change="onFilePicked"
      />

      <p style="width:100%; margin:10px 0 13px 0; background-color:gray; color:white;line-height:200%">
        &emsp;商品の説明を入力してください
      </p>
      <v-flex xs12>
        <v-textarea
          solo
          name="input-7-4"
          v-model="input"
          label=""
          value=""
          height=150
        ></v-textarea>
      </v-flex>
      <p style="width:100%; margin:10px 0 13px 0; background-color:gray; color:white;line-height:200%">
        &emsp;商品のカテゴリを追加してください
      </p>
      <v-layout row>
        <v-flex xs12>
          <v-text-field
            v-model="category"
            label=""
            solo
          ></v-text-field>
        </v-flex>
      </v-layout>
      
      <v-flex xs12>
          <v-btn color="red" round large type="submit" style="width:44%">出品する</v-btn>
          <v-btn color="blue" round large style="width:44%">キャンセル</v-btn>
      </v-flex>
      </form>
    </v-flex>
  </v-layout>
</template>

<script src="/******/v-preview-input/v-preview-input.js"></script>
<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'

  export default {
  
      fetch ({ store, route,redirect }) {
      console.log("今からリダイレクト分岐");
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
      text: [],  // 取得したメッセージを入れる配列
      input: '',  // 入力したメッセージ
      category: '',
      title: '',
      photo: null,
      photo_url: null,
      dialog: false,
      imageName: [],
      imageFile: [],
      uploadedImage: [],
      flag: "",
      imageUrl: [],
    }
    //console.log(user);
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

            // ファイルのパスを設定(1)
            var mountainsRef = storageRef.child(`images/${this.imageName[0]}`);
            // ファイルを適用してファイルアップロード開始
            mountainsRef.put(this.imageFile[0]).then(snapshot => {
              //パスを取得
              snapshot.ref.getDownloadURL().then(downloadURL =>{
              this.imageUrl.push(downloadURL);
              console.log("upload to "+this.imageName[0]);

                if(this.imageName[1] !== undefined){

                  // ファイルのパスを設定(2)
                  var mountainsRef = storageRef.child(`images/${this.imageName[1]}`);
                  // ファイルを適用してファイルアップロード開始
                  mountainsRef.put(this.imageFile[1]).then(snapshot => {
                    //パスを取得
                    snapshot.ref.getDownloadURL().then(downloadURL =>{
                    this.imageUrl.push(downloadURL);
                    console.log("upload to "+this.imageName[1]);

                      if(this.imageName[2] !== undefined){

                        // ファイルのパスを設定(3)
                        var mountainsRef = storageRef.child(`images/${this.imageName[2]}`);
                        // ファイルを適用してファイルアップロード開始
                        mountainsRef.put(this.imageFile[2]).then(snapshot => {
                          //パスを取得
                          snapshot.ref.getDownloadURL().then(downloadURL =>{
                          this.imageUrl.push(downloadURL);
                          console.log("upload to "+this.imageName[2]);

                          this.DBwriting(db,user);

                          });
                        });

                      }else{
                        this.DBwriting(db,user);
                      }
                  
                    });
                  });

                }else{
                  this.DBwriting(db,user);
                }

                

                });
              });
            

        })
      },
      pickFile() {
      this.$refs.image.click();
    },
    //ファイルの選択変えた時に動きそう
     onFilePicked(e) {
      const files = e.target.files;
      if (files[0] !== undefined) {
        this.imageName.push(files[0].name);
        const fr = new FileReader();
        fr.onload = e => {
          this.flag = "tinpo";
          this.uploadedImage.push(e.target.result);
        };
        fr.readAsDataURL(files[0]);
        fr.addEventListener("load", () => {
          this.imageFile.push(files[0]); // this is an image file that can be sent to server...
          console.log("pushed:"+this.imageName);
        });
      } 
    },

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
      },
      //書き込み処理
      DBwriting(db,user) {
        const itemInputData = {
        user_id: user.uid,
        user_photo:user.photoURL,
        user_name: user.displayName,
        item_name:this.title,
        item_text: this.input,
        category: this.category,
        image_url: this.imageUrl,
        created_at:new Date(),
      };
      //itemコレクションに対して
      db.collection('item').add(itemInputData)
      .then(docRef => {
        const usersInputData = {
        item_id: docRef.id,
        user_photo:user.photoURL,
        user_name: user.displayName,
        item_name:this.title,
        text: this.input,
        category: this.category,
        image_url: this.imageUrl,
        created_at:new Date(),
      };
      // ユーザーの出品一覧に対して
     db.collection("users/"+this.user.uid+"/item").doc().set(usersInputData)
      this.input = "";
      this.imageName= [],
      this.imageFile = [],
      this.uploadedImage = [],
      this.imageUrl = [],
      this.title = "",
      this.category = ""
      }).then(_ => {
        this.$router.push("/")
      });


    },
    


  },
  
};


</script>