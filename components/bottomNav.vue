 <template>
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
      
</template>