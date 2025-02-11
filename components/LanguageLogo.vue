<template>
  <router-link
    :to="`/${l1.code}/${l2.code}/`"
    class="link-unstyled d-inline-block"
  >
    <div class="logo-constructed">
      <div class="logo-circle-wrapper">
        <div
          :class="`logo-circle
        bg-gradient-${(l2.name || '').length.toString().split('').pop()}`"
        >
          <span class="logo-circle-initial">{{ l2Name.charAt(0) }}</span>
          <img
            v-if="l2.logo"
            :src="`/img/logo-square/${l2.code}.jpeg`"
            :alt="logoDescription"
            :title="logoDescription"
          />
        </div>
        <div
          class="logo-speech-bubble shadowed"
          :style="`background-image: url(/img/speech-light.png)`"
        >
          <b>{{ l2.code }}</b>
        </div>
      </div>
      <div class="logo-text text-white">
        <template>
          <div class="logo-text-language">
            {{ l2Name }}
          </div>
          <div class="logo-text-zth">
            <span v-if="!compact">
              {{
                l1.translations && l1.translations["Zero to Hero"]
                  ? l1.translations["Zero to Hero"]
                  : "Zero to Hero"
              }}
            </span>
            <span v-else>&nbsp;</span>
          </div>
        </template>
      </div>
    </div>
  </router-link>
</template>

<script>
import Config from "@/lib/config";

export default {
  props: ["l1", "l2", "compact"],
  data() {
    return {
      Config,
    };
  },
  computed: {
    l2Name() {
      let l2Name = this.l2.name.replace(/ \(.*\)/gi, "");
      l2Name =
        this.l1.translations && this.l1.translations[l2Name]
          ? this.l1.translations[l2Name]
          : l2Name;
      return l2Name;
    },
    logoDescription() {
      return this.l2.logoDesc
        ? `${this.l2.logoDesc}, a user of ${this.l2.name}.`
        : this.l2.name;
    },
  },
  methods: {},
};
</script>

<style lang="scss" scoped>
.logo-image {
  height: 4rem;
}
.logo-constructed {
  display: flex;
  align-items: flex-end;
  padding-top: 0.8rem;
  .logo-circle-wrapper {
    position: relative;
    margin-right: 0.7rem;
  }
  .logo-circle {
    height: 2.75rem;
    width: 2.75rem;
    border-radius: 100%;
    overflow: hidden;
    position: relative;
    box-shadow: 0 5px 10px rgba(0, 0, 0, 0.5);
    text-align: center;
    line-height: 2.75rem;
    .logo-circle-initial {
      color: white;
      font-size: 1.5rem;
      opacity: 0.7;
    }
    img {
      height: 100%;
      width: 100%;
      object-fit: cover;
      object-position: center;
      position: absolute;
      top: 0;
      left: 0;
    }
  }

  .logo-text * {
    font-family: "Helvetica Neue", Helvetica, sans-serif !important;
    text-align: left;
    margin-bottom: -0.4rem;
    text-shadow: 0 1px 8px rgba(0, 0, 0, 0.5);
  }
  .logo-speech-bubble {
    height: 2rem;
    width: 2rem;
    background-size: cover;
    line-height: 2rem;
    text-align: center;
    font-size: 0.7rem;
    color: #555;
    position: absolute;
    top: -0.9rem;
    right: -0.4rem;
  }
  .logo-speech-bubble b {
    display: block;
    position: relative;
    bottom: 0.1em;
  }
  .logo-text-language {
    font-weight: 200;
    letter-spacing: 0.1em;
    margin-bottom: -0.2em;
    text-transform: uppercase;
    line-height: 1.15;
  }
  .logo-text-zth {
    font-weight: bold;
    text-transform: uppercase;
  }
}
</style>