<template>
  <v-layout>
    <v-flex v-if="loading">
      <h2 class="l-single-head">お土産の情報を入力</h2>
      <form action @submit.prevent="sendMessage" class="form">
      
        <p style="margin-top:0px;color:#333333;font-size:16px;font-weight:500">
          &emsp;出品画像<span class="form-require">必須</span>
        </p><p style="margin-top:-5px;font-size:14px">&emsp;最大3枚までアップロードできます</p>
      
      <v-flex xs12 sm6 md3 style="background-color:white">
<v-container grid-list-md text-xs-center>
        <v-layout row wrap>

          <!-- 画像一つ目 -->
          <v-flex xs4 v-show="uploadedImage[0]">
            <v-img
              :src="uploadedImage[0]"
            >
              <v-btn
                fab dark small color="primary"
                @click="remove(0)"
              >
                <v-icon>clear</v-icon>
              </v-btn>
            </v-img>
          </v-flex>

          <!-- 画像二つ目 -->
          <v-flex xs4 v-show="uploadedImage[1]">
            <v-img
              :src="uploadedImage[1]"
            >
              <v-btn
                fab dark small color="primary"
                @click="remove(1)"
              >
                <v-icon>clear</v-icon>
              </v-btn>
            </v-img>
          </v-flex>

          <!-- 画像三つ目 -->
          <v-flex xs4 v-show="uploadedImage[2]">
            <v-img
              :src="uploadedImage[2]"
            >
              <v-btn
                fab dark small color="primary"
                @click="remove(2)"
              >
                <v-icon>clear</v-icon>
              </v-btn>
            </v-img>
          </v-flex>

        </v-layout>
      </v-container>

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
        id="file"
        multiple="multiple"
        type="file"
        style="display: none"
        ref="image"
        accept="image/*"
        @change="onFilePicked"
      />

      <v-dialog v-model="error_dialog" scrollable max-width="300px">
        <v-card>
          <v-card-text>{{Message}}</v-card-text>
          <v-divider></v-divider>
        </v-card>
      </v-dialog>
      </v-flex>

      <p style="margin-top:20px;margin-bottom:0px;color:#333333;font-size:16px;font-weight:500">
        &emsp;お土産名と説明
        <span class="form-require">必須</span>
      </p>
      <v-flex xs12 sm6 md3 style="background-color:white">
        <v-container>
            <!-- success -->
          <v-text-field
            label="商品名"
            placeholder="(必須、40文字まで)"
            v-model="title"
            color="blue"
          ></v-text-field>

          <v-textarea
            label="商品の説明"
            placeholder="(任意、1000文字まで)
  (色、素材、重さ、定価、注意点など)"
            v-model="input"
            value=""
            height=150
            color="blue"
          ></v-textarea>
        </v-container>
      </v-flex>
      <v-flex xs12 sm6 d-flex style="background-color:white">
        <v-container>
          <v-select
            :items="categoryItems"
            label="カテゴリー"
            v-model="category"
          ></v-select>
        </v-container>
      </v-flex>

      <v-flex xs12 sm6 d-flex style="background-color:white">
        <v-container>
    <v-dialog v-model="dialog" scrollable max-width="300px">
      <template v-slot:activator="{ on }">
        <v-btn color="primary" dark v-on="on">国名選択</v-btn>
      </template>
      <v-card>
        <v-card-title>国名選択</v-card-title>
        <v-divider></v-divider>
        <v-card-text style="height: 300px;">
          <v-radio-group v-model="country" column>
            <v-radio v-for="item in countryItems" :key="item" :label="item" :value="item"></v-radio>
          </v-radio-group>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-btn color="blue darken-1" text @click="dialog = false">キャンセル</v-btn>
          <v-btn color="blue darken-1" text @click="dialog = false">決定</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
            {{country}}
        </v-container>
      </v-flex>

      <v-layout justify-center>
        <v-btn color="red" round large type="submit" style="width:70%;height:60px;color:white">出品する</v-btn>
      </v-layout>
      <br/><br/>
      </form>
    </v-flex>
          <v-progress-circular
      v-else
      :size="70"
      :width="7"
      color="purple"
      indeterminate
      style="position:absolute;margin-left:40%;margin-top:100px"
    ></v-progress-circular>
  </v-layout>
</template>

<script src="/******/v-preview-input/v-preview-input.js"></script>
<script>
import createPersistedState from 'vuex-persistedstate'
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters } from 'vuex'
import uuid from 'uuid'

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
      Message: '',
      photo: null,
      photo_url: null,
      dialog: false,
      error_dialog: false,
      imageName: [],
      imageFile: [],
      uploadedImage: [],
      flag: "",
      imageUrl: [],
      loading: true,
      categoryItems: ['レディース', 'メンズ', 'ベビー・キッズ', 'インテリア・住まい・小物','食品','本・音楽・ゲーム','おもちゃ・ホビー・グッズ','コスメ・香水・美容','家電・スマホ・カメラ','スポーツ・レジャー','ハンドメイド','チケット','自動車・オートバイ','その他'],
      countryItems:['アメリカ合衆国','中華人民共和国','大韓民国','台湾','タイ','シンガポール','スペイン','ベトナム','香港','インドネシア','ドイツ','その他'],
      country:''
    }
    //console.log(user);
  },
    methods : {
      ...mapActions(['setUser']),
      sendMessage(){
        this.loading = false;
        firebase.auth().onAuthStateChanged(user => {
            this.user = user ? user : {}
            //console.log(this.user.uid)
            const db = firebase.firestore()
            // ストレージオブジェクト作成
            var storageRef = firebase.storage().ref();

            // ファイルのパスを設定(1)
            this.imageName[0] = uuid();
            var mountainsRef = storageRef.child(`images/${this.imageName[0]}`);
            // ファイルを適用してファイルアップロード開始
            mountainsRef.put(this.imageFile[0]).then(snapshot => {
              //パスを取得
              snapshot.ref.getDownloadURL().then(downloadURL =>{
              this.imageUrl.push(downloadURL);
              console.log("upload to "+this.imageName[0]);

                if(this.imageName[1] !== undefined){

                  // ファイルのパスを設定(2)
                  this.imageName[1] = uuid();
                  var mountainsRef = storageRef.child(`images/${this.imageName[1]}`);
                  // ファイルを適用してファイルアップロード開始
                  mountainsRef.put(this.imageFile[1]).then(snapshot => {
                    //パスを取得
                    snapshot.ref.getDownloadURL().then(downloadURL =>{
                    this.imageUrl.push(downloadURL);
                    console.log("upload to "+this.imageName[1]);

                      if(this.imageName[2] !== undefined){

                        // ファイルのパスを設定(3)
                        this.imageName[2] = uuid();
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
        var obj = document.getElementById("file");
        obj.value = "";
        if(this.imageName[2] !== undefined){
          //画像が三つアップロードされている場合警告
          this.Message = "画像は三つ以上選択できません"
          this.error_dialog = true;
        }else{
          this.$refs.image.click();
        }
      },
      //ファイルの選択変えた時に動きそう
      onFilePicked(e) {
        var files = e.target.files;
    　　//同名のファイルが選択されたときにエラーを吐かせる
        var sameNameCheck = false;
        for(var i = 0;i < files.length;i++){
          for(var j = 0;j < this.imageName.length;j++){
            if(files[i].name === this.imageName[j]){
              sameNameCheck = true;
            }
          }
        }
        //画像アップロード時にファイルが四つ以上選択された場合警告
        if(files.length + this.imageName.length > 3){
          this.Message = "画像は三つ以上選択できません";
          this.error_dialog = true;
        }else if(sameNameCheck){
          this.Message = "同じ名前のファイルは選択できません";
          this.error_dialog = true;
              console.log("nakami:"+this.imageFile);
        }else{
          for (var i = 0, f; f = files[i]; i++) {
            this.imageName.push(files[i].name);
            this.imageFile.push(files[i]); // this is an image file that can be sent to server...
            var fr = new FileReader;
            fr.readAsDataURL(f);
            fr.onload = e => {
              this.uploadedImage.push(e.target.result);
            };
            fr.addEventListener("load", () => {
              console.log("nakami:"+this.imageName);
              console.log("nakami:"+this.imageFile);
            });
          }
        }
      },
      remove(number) {
        this.imageFile.splice(number,1);
        this.imageName.splice(number,1);
        this.uploadedImage.splice(number,1);
        this.imageUrl.splice(number,1);
        console.log("nakami:"+this.imageName);
        console.log("nakami:"+this.imageFile);
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
        display_flg:true,
        country:this.country,
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
          display_flg:true,
          country:this.country,
          created_at:new Date(),
        };
        // ユーザーの出品一覧に対して
        db.collection("users/"+this.user.uid+"/item").doc(docRef.id).set(usersInputData)
        this.input = "";
        this.imageName= [],
        this.imageFile = [],
        this.uploadedImage = [],
        this.imageUrl = [],
        this.title = "",
        this.category = ""
      }).then(_ => {
        this.$router.push("/")
        this.loading = true;
      });


    },
    


  },
  
};


</script>
<style>
.form-require{
    margin: 0 0 0 8px;
    padding: 2px 4px;
    border-radius: 2px;
    color: #fff;
    font-size: 12px;
    vertical-align: top;
    background: #ea352d;
}

.l-single-head {
    padding: 24px 8px;
    font-size: 18px;
    line-height: 1.5;
    text-align: center;
}
</style>