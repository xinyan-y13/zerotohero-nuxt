<template>
  <container-query :query="query" v-model="params">
    <div>
      <ul v-if="languages && languages.length > 0" :class="classes">
        <li
          v-for="(language, index) in languages"
          :key="`lang-${language.code}-${index}`"
          class="language-list-item"
          :set="(base = languagePath(language))"
        >
          <span class="language-list-item-features">
            <router-link
              :to="`${base}live-tv/`"
              :class="{
                'feature-icon mr-1': true,
                transparent: !hasLiveTV(english, language),
              }"
              title="Live TV"
            >
              <i class="fa fa-broadcast-tower" />
            </router-link>
            <router-link
              :to="`${base}youtube/browse/all/all/0`"
              :class="{
                'feature-icon mr-1': true,
                transparent: !hasYouTube(english, language),
              }"
              title="Videos"
            >
              <i class="fa fa-play-circle" />
            </router-link>
            <router-link
              :to="`${base}dictionary`"
              :class="{
                'feature-icon mr-1': true,
                transparent: !hasDictionary(english, language),
              }"
              title="Dictionary"
            >
              <i class="fas fa-book" />
            </router-link>
            <span
              v-if="
                variant === 'icon' &&
                showSpeakers &&
                language.speakers &&
                language.speakers > 0
              "
              class="language-list-item-speakers"
            >
              {{ speakers(language.speakers) }}
            </span>
          </span>
          <router-link :to="base" class="language-list-item-name">
            {{ languageName(language) }}
            <span v-if="showCode">({{ language.code }})</span>
          </router-link>
          <span
            v-if="
              variant === 'list' &&
              showSpeakers &&
              language.speakers &&
              language.speakers > 0
            "
            class="language-list-item-speakers"
          >
            {{ speakers(language.speakers) }}
          </span>
        </li>
      </ul>
    </div>
  </container-query>
</template>

<script>
import Helper from "@/lib/helper";
import { ContainerQuery } from "vue-container-query";
export default {
  components: {
    ContainerQuery,
  },
  props: {
    langs: {
      default: undefined,
    },
    codes: {
      default: undefined,
    },
    sort: {
      default: false,
    },
    singleColumn: {
      default: false,
    },
    showSpeakers: {
      default: true,
    },
    showCode: {
      default: false,
    },
    skin: {
      default: "light",
    },
    variant: {
      default: "list", // or 'icon'
    },
  },
  data() {
    return {
      specials: Helper.specialLanguages,
      params: {},
      query: {
        xs: {
          minWidth: 0,
          maxWidth: 423,
        },
        sm: {
          minWidth: 423,
          maxWidth: 720,
        },
        md: {
          minWidth: 720,
          maxWidth: 960,
        },
        lg: {
          minWidth: 960,
          maxWidth: 1140,
        },
        xl: {
          minWidth: 1140,
        },
      },
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
    classes() {
      let classes = {
        "language-list": true,
        "language-list-single-column": this.singleColumn,
        "language-list-1-col": this.params.xs,
        "language-list-2-cols": this.params.sm || this.params.md,
        "language-list-3-cols": this.params.lg,
        "language-list-4-cols": this.params.xl,
      };
      classes[`language-list-${this.skin}`] = true;
      classes[`language-list-${this.variant}`] = true;
      return classes;
    },
    english() {
      return this.$languages.l1s.find((language) => language.code === "en");
    },
    languages() {
      let languages;
      if (this.langs) {
        languages = this.langs.filter((l) => l);
      }
      if (!languages && this.codes) {
        languages = this.codes
          .map((c) => this.$languages.getSmart(c))
          .filter((l) => l);
      }
      if (this.sort) {
        languages = Helper.uniqueByValue(languages, "iso639-3").sort((a, b) =>
          this.languageName(a).localeCompare(this.languageName(b), "en")
        );
      }
      return languages;
    },
  },
  methods: {
    speakers(number) {
      return Helper.formatK(number, 1);
    },
    languagePath(language) {
      let l1 = "en";
      let special = this.specials[language.code];
      if (special) l1 = special.l1;
      return `/${l1}/${language.code}/`;
    },
    languageName(language) {
      let name = language.name.replace(/ \(.*\)/gi, "");
      let special = this.specials[language.code];
      if (special) name = special.name;
      return name;
    },
    hasDictionary(l1, l2) {
      return (
        this.$languages.hasFeature(l1, l2, "dictionary") || l2.code === "en"
      );
    },
    hasYouTube(l1, l2) {
      return this.$languages.hasYouTube(l1, l2);
    },
    hasLiveTV(l1, l2) {
      return this.$languages.hasFeature(l1, l2, "live-tv");
    },
  },
};
</script>

<style lang="scss" scoped>
.language-list {
  list-style: none;
  padding: 0;
  column-gap: 2rem;
  margin-bottom: 0;
  .language-list-item {
    .language-list-item-speakers {
      font-size: 0.8em;
      white-space: nowrap;
      margin-left: 0.25rem;
    }
  }

  &.language-list-light {
    .language-list-item {
      a {
        color: hsla(14deg 98% 10% / 80%);
      }
      .feature-icon {
        color: hsla(14deg 98% 10% / 60%);
      }
      .language-list-item-speakers {
        color: hsla(14deg 98% 10% / 40%);
      }
    }
  }
  &.language-list-dark {
    .language-list-item {
      a {
        color: rgba(255, 255, 255, 0.8);
      }
      .feature-icon {
        color: rgba(255, 255, 255, 0.6);
      }
      .language-list-item-speakers {
        color: rgba(255, 255, 255, 0.4);
      }
    }
  }
  &.language-list-icon {
    .language-list-item {
      display: table;
      width: 100%;
      .language-list-item-name {
        display: table-header-group;
      }
      .language-list-item-features {
        display: table-footer-group;
        line-height: 0.8;
        .feature-icon {
          font-size: 0.8em;
          &.transparent {
            display: none;
          }
        }
        .language-list-item-speakers {
          margin: 0;
        }
      }
    }
  }
}

.language-list:not(.language-list-single-column) {
  &.language-list-1-col {
    column-count: 1;
  }
  &.language-list-2-cols {
    column-count: 2;
  }
  &.language-list-3-cols {
    column-count: 3;
  }
  &.language-list-4-cols {
    column-count: 4;
  }
}
</style>