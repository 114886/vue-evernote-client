<template>
  <div id="note" class="detail">
    <note-sidebar @update:notes="(val) => (notes = val)"></note-sidebar>
    <div class="note-detail">
      <div class="note-empty" v-show="!curBook.id">请创建笔记本后</div>
      <div class="note-empty" v-show="!curNote.id">选择或创建笔记</div>
      <div class="note-detail-ct" v-show="curNote.id">
        <div class="note-bar">
          <span> 创建日期: {{ curNote.createdAtFriendly }}</span>
          <span> 更新日期: {{ curNote.updatedAtFriendly }}</span>
          <span> {{ statusText }}</span>
          <span class="iconfont icon-delete" @click="onDeleteNote"></span>
          <span
            class="iconfont"
            :class="isShowPreview ? 'icon-yulanmoshi' : 'icon-bianjimoshi'"
            @click="isShowPreview = !isShowPreview"
          ></span>
          <span v-show="isShowPreview === false">编辑模式</span>
          <span v-show="isShowPreview === true">预览模式</span>
        </div>
        <div class="note-title">
          <input
            type="text"
            v-model="curNote.title"
            @input="onUpdateNote"
            @keydown="statusText = '正在输入...'"
            placeholder="输入标题"
          />
        </div>
        <div class="editor">
          <codemirror
            v-model="curNote.content"
            :options="cmOptions"
            v-show="!isShowPreview"
            @input="onUpdateNote"
            @inputRead="statusText = '正在输入...'"
          ></codemirror>
          <!--  <textarea v-show="isShowPreview"  v-model="curNote.content" @input="onUpdateNote" @keydown="statusText='正在输入...'" placeholder="输入内容, 支持 markdown 语法"></textarea>-->
          <div
            class="preview markdown-body"
            v-html="previewContent"
            v-show="isShowPreview"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import NoteSidebar from "@/components/NoteSidebar";
import _ from "lodash";
import MarkdownIt from "markdown-it";
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";
import { codemirror } from "vue-codemirror";
import "codemirror/lib/codemirror.css";
import "codemirror/mode/markdown/markdown.js";
import "codemirror/theme/neat.css";

let md = new MarkdownIt();

export default {
  components: {
    NoteSidebar,
    codemirror,
  },

  data() {
    return {
      statusText: "笔记未改动",
      isShowPreview: false,
      cmOptions: {
        tabSize: 4,
        mode: "text/x-markdown",
        theme: "neat",
        lineNumbers: false,
        line: true,
      },
    };
  },

  created() {
    this.checkLogin({ path: "/login" });
  },

  computed: {
    ...mapGetters(["notes", "curNote", "curBook"]),

    previewContent() {
      return md.render(this.curNote.content || "");
    },
  },

  methods: {
    ...mapMutations(["setCurNote"]),

    ...mapActions(["updateNote", "deleteNote", "checkLogin"]),

    onUpdateNote: _.debounce(function () {
      if (!this.curNote.id) return;
      this.updateNote({
        noteId: this.curNote.id,
        title: this.curNote.title,
        content: this.curNote.content,
      })
        .then((data) => {
          this.statusText = "已保存";
        })
        .catch((data) => {
          this.statusText = "保存出错";
        });
    }, 1000),

    onDeleteNote() {
      this.deleteNote({ noteId: this.curNote.id }).then((data) => {
        this.$router.replace({ path: "/note" });
      });
    },
  },

  beforeRouteUpdate(to, from, next) {
    this.setCurNote({ curNoteId: to.query.noteId });
    next();
  },
};
</script>

<style lang="less">
@import url(../assets/css/note-detail.less);

#note {
  display: flex;
  align-items: stretch;
  background-color: #fff;
  flex: 1;
}
</style>