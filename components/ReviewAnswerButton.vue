<template>
  <button
    :class="{
      'review-answer': true,
      'review-answer-light': skin === 'light',
      'review-answer-dark': skin === 'dark',
      btn: true,
      'bg-white': skin === 'light',
      'btn-ghost-dark': skin === 'dark',
      checked: checked,
      'review-answer-correct': answer.correct,
    }"
    @click="answered()"
  >
    <template v-if="$l2.code === 'ja' || !$l2.han">
      {{ answer.text }}
    </template>
    <template v-else-if="$l2.han && $useTraditional">
      {{ answer.traditional || answer.simplified }}
    </template>
    <template v-else-if="$l2.han && !$useTraditional">
      {{ answer.simplified || answer.traditional }}
    </template>
  </button>
</template>

        
<script>
export default {
  props: {
    answer: {
      type: Object,
    },
    skin: {
      default: "light",
    },
  },
  data() {
    return {
      checked: false,
    };
  },
  methods: {
    answered() {
      this.checked = true;
      this.$emit("answered");
    },
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
    $useTraditional() {
      if (typeof this.$store.state.settings.l2Settings !== "undefined")
        return this.$store.state.settings.l2Settings.useTraditional;
    },
  },
};
</script>
        
<style lang="scss" scoped>
.review-answer,
.review-answer:hover {
  &.review-answer-light {
    border: 1px solid #999;
    color: #999;
  }
  padding: 0.1rem 0.3rem;
  font-size: 0.9em;
  min-width: 5rem;
  margin-right: 0.5rem;
}

.review-answer.checked:not(.review-answer-correct) {
  background-color: #dc3838 !important;
  border-color: #a03030 !important;
  color: white !important;
}

.review-answer.checked.review-answer-correct {
  background-color: #63ab67 !important;
  border-color: #36823b !important;
  color: white !important;
}
</style>