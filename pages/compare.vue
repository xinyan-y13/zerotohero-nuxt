<router>
  {
    path: '/:l1/:l2/compare/:method/:args',
  }
</router>
<template>
  <div class="main" v-cloak>
    <SocialHead :title="title" :description="description" :image="image" />
    <div class="container pt-4 pb-4 focus">
      <div class="row">
        <div class="col-12">
          <SearchCompare :searchEntry="a" :compareEntry="b" :compare="true" />
        </div>
      </div>
      <div class="row mt-4 mb-3">
        <div class="col-6">
          <div class="text-center">
            <Loader v-if="!a" class="mt-5" />
          </div>
          <LazyEntryHeader
            v-if="a"
            :entry="a"
            class="text-center"
            :key="`${a.id}-header`"
            @prevWord="prevWord()"
            @nextWord="nextWord()"
          />
        </div>
        <div class="col-6">
          <div class="text-center">
            <Loader v-if="!b" class="mt-5" />
          </div>
          <LazyEntryHeader
            v-if="b"
            :entry="b"
            class="text-center"
            :key="`${b.id}-header`"
          />
        </div>
      </div>

      <div class="row">
        <div class="col-sm-12">
          <LazyCompareDefs
            v-if="a && b"
            :a="a"
            :b="b"
            :key="`${a.id}-${b.id}-defs`"
          />
        </div>
      </div>
    </div>
    <div
      class="jumbotron-fluid focus"
      v-if="a && b && a.example && b.example"
    >
      <div class="container">
        <div class="row">
          <div class="col-sm-6 mb-5">
            <LazyEntryExample
              :key="`${a.id}-example`"
              :entry="a"
              id="compare-example-a"
            />
          </div>
          <div class="col-sm-6 mb-5">
            <LazyEntryExample
              :key="`${b.id}-example`"
              :entry="b"
              id="compare-example-b"
            />
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <div class="row">
        <div class="col-md-12">
          <div
            class="widget widget-dark"
            v-if="a && b"
            :key="`${a.id}-subs`"
            id="compare-search-subs"
          >
            <div class="widget-title">
              “{{ a.bare }}” and “{{ b.bare }}” in TV Shows
            </div>
            <div class="widget-body">
              <LazyCompareSearchSubs
                skin="dark"
                :key="`compare-search-subs-${a.id}-${b.id}`"
                :levelA="a.newHSK && a.newHSK === '7-9' ? '7-9' : a.hsk || a.level || 'outside' "
                :termsA="
                  ['zh', 'yue'].includes($l2.code)
                    ? a.simplified === a.traditional
                      ? [a.simplified]
                      : [a.simplified, a.traditional]
                    : [a.bare]
                "
                :levelB="b.newHSK && b.newHSK === '7-9' ? '7-9' : b.hsk || a.level || 'outside'"
                :termsB="
                  ['zh', 'yue'].includes($l2.code)
                    ? b.simplified === b.traditional
                      ? [b.simplified]
                      : [b.simplified, b.traditional]
                    : [b.bare]
                "
              />
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <div class="row mt-5">
        <div class="col-sm-6">
          <LazyWebImages
            v-if="a"
            :text="a.bare"
            limit="10"
            :preloaded="aImages"
            :key="`${a.id}-images`"
          />
        </div>
        <div class="col-sm-6">
          <LazyWebImages
            v-if="b"
            :text="b.bare"
            limit="10"
            :preloaded="bImages"
            :key="`${b.id}-images`"
          />
        </div>
      </div>
    </div>

    <div class="container">
      <div class="row">
        <div class="col-sm-12">
          <LazyCompareCollocations
            class="mt-5 focus"
            v-if="a && b"
            :term="a.bare"
            :compareTerm="b.bare"
            :level="a.level"
            :compareLevel="b.level"
          />
        </div>
      </div>
    </div>

    <div class="container">
      <div class="row">
        <div class="col-sm-12 mt-5" v-if="a">
          <LazyEntryRelated :key="`${a.id}-related`" :entry="a" />
        </div>
        <div class="col-sm-12 mt-5" v-if="b">
          <LazyEntryRelated :key="`${a.id}-related`" :entry="b" />
        </div>
      </div>
    </div>

    <!-- <EntryCharacters :entry="entry"></EntryCharacters> -->
    <div class="container pt-5 pb-5 focus">
      <div class="row">
        <div class="col-sm-6">
          <LazyConcordance
            v-if="a"
            :text="a.bare"
            :level="a.hsk"
            :key="`${a.id}-concordance`"
          />
        </div>
        <div class="col-sm-6">
          <LazyConcordance
            v-if="b"
            :text="b.bare"
            :level="b.hsk"
            :key="`${b.id}-concordance`"
          />
        </div>
      </div>
    </div>

    <LazyEntryCourseAd
      v-if="$l2 === 'zh' && a && b"
      :entry="b.hsk > a.hsk ? b : a"
      :key="`${a.id}-${b.id}-ad`"
    />
  </div>
</template>

<script>
import WordPhotos from '@/lib/word-photos'

export default {
  data() {
    return {
      a: undefined,
      b: undefined,
      aKey: 0,
      bKey: 100,
      aImages: [],
      bImages: [],
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
    title() {
      if (this.a && this.b) {
        return `“${this.a.bare}” vs “${this.b.bare}” - ${
          this.$l2 ? this.$l2.name : ""
        } Words Compared | ${this.$l2 ? this.$l2.name : ""} Zero to Hero`;
      }
      return `${this.$l2 ? this.$l2.name : ""} Words Compared | ${
        this.$l2 ? this.$l2.name : ""
      } Zero to Hero`;
    },
    description() {
      if (this.a && this.b) {
        return `See how the two ${this.$l2 ? this.$l2.name : ""} words “${
          this.a.bare
        }” and “${
          this.b.bare
        }” are used differently in common collocations and on TV shows.`;
      }
      return `Compare two  ${
        this.$l2 ? this.$l2.name : ""
      } words and see how they are used differently in common collocations and on TV shows.`;
    },
    image() {
      if (this.aImages.length > 0 || this.bImages.length > 0) {
        return this.bImages.length > 0 ? this.bImages[0].src : this.aImages[0].src;
      } else {
        return "/img/zth-share-image.jpg";
      }
    },
  },
  async fetch() {
    let method = this.$route.params.method;
    let args = this.$route.params.args.split(",");
    let aId = args[0];
    let bId = args[1];
    if (args.length === 6) {
      // When we use hsk-cedict for chinese
      aId = [args[0], args[1], args[2]].join(",");
      bId = [args[3], args[4], args[5]].join(",");
    }
    if (method && args) {
      if (method === "hsk") {
        this.a = await (await this.$getDictionary()).getByHSKId(aId);
        this.b = await (await this.$getDictionary()).getByHSKId(bId);
      } else if (method === "bare") {
        let resultsA = await (await this.$getDictionary()).lookupbare(aId);
        this.a = resultsA[0];
        let resultsB = await (await this.$getDictionary()).lookupbare(bId);
        this.b = resultsB[0];
      } else if (method === "simplified") {
        let resultsA = await (await this.$getDictionary()).lookupSimplified(
          args[0]
        );
        this.a = resultsA[0];
        let resultsB = await (await this.$getDictionary()).lookupSimplified(
          args[1]
        );
        this.b = resultsB[0];
      } else if (method === "traditional") {
        let resultsA = await (await this.$getDictionary()).lookupTraditional(
          args[0]
        );
        this.a = resultsA[0];
        let resultsB = await (await this.$getDictionary()).lookupTraditional(
          args[1]
        );
        this.b = resultsB[0];
      } else {
        this.a = await (await this.$getDictionary()).get(aId);
        this.b = await (await this.$getDictionary()).get(bId);
      }
    }
    this.aImages = await WordPhotos.getGoogleImages({
      term: this.a.bare,
      lang: this.$l2.code,
    });
    this.bImages = await WordPhotos.getGoogleImages({
      term: this.b.bare,
      lang: this.$l2.code,
    });
  },
  methods: {
    unbindKeys() {
      window.onkeydown = null;
    },
    bindKeys() {
      window.onkeydown = (e) => {
        if (
          !["INPUT", "TEXTAREA"].includes(e.target.tagName.toUpperCase()) &&
          !e.metaKey
        ) {
          if (e.keyCode == 36) {
            // home
            document
              .getElementById("main")
              .scrollIntoView({ behavior: "smooth" });
            // this.$refs.searchCompare.focusOnSearch()
            e.preventDefault();
            return false;
          }
          if (e.keyCode == 35) {
            // end
            document
              .getElementById("compare-search-subs")
              .scrollIntoView({ behavior: "smooth" });
            e.preventDefault();
            return false;
          }
        }
      };
    },
  },
  activated() {
    this.bindKeys();
  },
  deactivated() {
    this.unbindKeys();
  },
};
</script>
