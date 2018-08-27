<template>
  <div class="hello">
    <c-title :text="title"></c-title>
    <img class="logo" src="../assets/images/logo.png">
    <p class="welcome">{{message}} by {{author}}</p>
    <div v-html="content"></div>

    <div>
      <!--<Ueditor :value="ueditor.value" :config="ueditor.config" ref="ue"></Ueditor>
      <input type="button" value="显示编辑器内容（从控制台查看）" @click="returnContent">-->

      <VueUEditor @ready="editorReady" style="width: 800px"></VueUEditor>
      <VueUEditor :ueditorConfig="editorConfig" style="width: 800px"></VueUEditor>

    </div>
  </div>
</template>

<script>

  import {mapState} from 'vuex';
  import cTitle from 'components/title';
  // import Ueditor from "components/uev1";
  import VueUEditor from "components/uev1";

  export default {
    data () {
      return {
        title: 'Hello Vue!',
        content: '',
        dat: {
            content: "",
        },
        ueditor: {
            value: "编辑默认文字",
            config: {}
        },
          editorConfig: {
              toolbars: [[
                  'fullscreen', 'source', '|', 'undo', 'redo', '|',
                  'bold', 'italic', 'underline', 'fontborder', 'strikethrough', 'superscript', 'subscript', 'removeformat', 'formatmatch', 'autotypeset', 'blockquote', 'pasteplain', '|', 'forecolor', 'backcolor', 'insertorderedlist', 'insertunorderedlist', 'selectall', 'cleardoc', '|',
                  'skip'
              ]]
          }
      }
    },
    methods: {
      async getContent () {
        const response = await fetch('/api/hello');
        this.content = await response.text();
      },
      returnContent() {
          this.dat.content = this.$refs.ue.getUEContent();
          console.log(this.dat.content);
      },
      showContent() {
          this.show = !this.show;
      },
      editorReady (editorInstance) {
          editorInstance.setContent('Hello world!<br>你可以在这里初始化编辑器的初始内容。');
          editorInstance.addListener('contentChange', () => {
              console.log('编辑器内容发生了变化：', editorInstance.getContent());
          });
      }
    },
    mounted () {
      this.$store.commit('message', '欢迎使用 vue！');
      this.getContent();
    },
    computed: mapState({
        message: state => state.message,
        author: state => state.author
    }),
    components: {cTitle, VueUEditor}
  }

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" rel="stylesheet/scss" scoped>
  h1 {
    color: #42b983;
  }

  .logo {
    width: 100px;
    height: 100px;
    a {
      color: #42b983;
      text-decoration: none;
    }
  }
</style>
