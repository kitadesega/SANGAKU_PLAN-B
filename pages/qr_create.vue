<template>
<qriously value="" :size="200" />
</template>

<script>
import Vue from 'vue'
import VueQriously from 'vue-qriously'
Vue.use(VueQriously)

export default {
  components: {
  },
  name: "qrGenarate", 
  data() {
    return {
      config: {
        // valueにはinput v-modelにて動的に入力した値が設定されるため空文字を設定
        value: "", 
        imagePath: "/img/example.jpg",
        filter: "color",
      }
    };
  },
}
</script>