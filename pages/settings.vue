<router>
  {
    path: '/:l1/:l2/settings',
    meta: {
      title: 'Settings | Zero to Hero',
      metaTags: [
        {
          name: 'description',
          content: 'Change preferences: choose a different text corpus.'
        }
      ]
    }
  }
</router>
<template>
  <div class="main pt-5 pb-5" v-cloak>
    <div class="container">
      <div class="row">
        <div class="col-sm-12">
          <h3 class="page-title text-center">
            {{ $t("Settings") }}
          </h3>
          <div class="jumbotron mt-4 p-4 bg-warning text-dark text-center">
            Settings are
            <b>automatically saved</b>
            as soon as you make the change.
          </div>
        </div>
      </div>
    </div>
    <div class="container">
      <div class="row">
        <div class="col-sm-12">
          <h5>General settings</h5>
          <b-form-checkbox v-model="subsSearchLimit">
            Limit "this word in TV Shows" search result (faster)
          </b-form-checkbox>
          <hr />
          <h5>Settings specific to {{ $l2.name }}:</h5>
          <AnnotationSettings />
          <div>
            <h5>{{ $t("Text Corpus Settings") }}</h5>
            <p v-if="$l1.code === 'zh'">
              <a
                href="https://baike.baidu.com/item/%E8%AF%AD%E6%96%99%E5%BA%93"
                target="_blank"
              >
                语料库
              </a>
              是用大量语言文本编写而成的集合，让我们我们可以提取词语搭配和例句。我们的文本语料库由
              <a href="https://www.sketchengine.eu/">Sketch Engine</a>
              提供，有许多不同的英语语料库可供选择。根据你选择的语料库，你看到的例句和词语搭配会有所不同。
            </p>
            <p v-else>
              A
              <a
                href="https://en.wikipedia.org/wiki/Text_corpus"
                target="_blank"
              >
                text corpus
              </a>
              is a large collection of text written in a language, where we can
              extract collocations and example sentences.
              <a href="https://www.sketchengine.eu/">Sketch Engine</a>
              , our text corpora provider, has a number of {{ $l2.name }} text
              corpora to select from. Depending on the corpus you choose, the
              example sentences and collocations you see will be different.
            </p>
          </div>
        </div>
      </div>
    </div>
    <div class="container-fluid">
      <CorpusSelect />
    </div>
    <div class="container">
      <div class="row">
        <div class="col-sm-12">
          <b-form-checkbox
            v-model="adminMode"
            style="display: inline-block"
            v-if="adminMode"
          >
            Admin mode
          </b-form-checkbox>
          <b-input-group v-else>
            <b-form-input
              v-model.lazy="adminModePasscode"
              placeholder="Enter passcode to enable admin mode."
            />
            <b-input-group-append>
              <b-button variant="primary">Enable</b-button>
            </b-input-group-append>
          </b-input-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CorpusSelect from "@/components/CorpusSelect";
import AnnotationSettings from "@/components/AnnotationSettings";
import { mapState } from "vuex";

export default {
  components: {
    CorpusSelect,
    AnnotationSettings,
  },
  mounted() {
    if (typeof this.$store.state.settings !== 'undefined') {
      this.subsSearchLimit = this.$store.state.settings.subsSearchLimit;
      this.adminMode = this.$store.state.settings.adminMode;
    }
    this.unsubscribe = this.$store.subscribe((mutation, state) => {
      if (mutation.type === "settings/LOAD_SETTINGS") {
        this.subsSearchLimit = this.$store.state.settings.subsSearchLimit;
        this.adminMode = this.$store.state.settings.adminMode;
      }
    });
  },
  data() {
    return {
      subsSearchLimit: true,
      adminMode: false,
      adminModePasscode: "",
    };
  },
  computed: {
    $l1() {
      if (typeof this.$store.state.settings.l1 !== "undefined")
        return this.$store.state.settings.l1;
    },
    $l2() {
      if (typeof this.$store.state.settings.l2 !== "undefined")
        return this.$store.state.settings.l2;
    },
    ...mapState("settings", ["l2Settings", "l1", "l2"]),
  },
  watch: {
    subsSearchLimit() {
      this.$store.commit(
        "settings/SET_SUBS_SEARCH_LIMIT",
        this.subsSearchLimit
      );
    },
    adminMode() {
      this.$store.commit("settings/SET_ADMIN_MODE", this.adminMode);
    },
    adminModePasscode() {
      if (this.adminModePasscode === "A lion flies on camera") {
        this.adminMode = true;
      }
    },
  },
};
</script>