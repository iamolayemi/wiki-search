<template>
  <div class="result">
    <h3>{{ result.title }}</h3>
    <p v-html="result.snippet"></p>
    <a :href="url">Read More</a>
  </div>
</template>

<script>
import { computed } from "@vue/runtime-core";

export default {
  name: "Result",
  props: ["result"],

  setup(props) {
    const url = computed(() => {
      return `https://en.wikipedia.org/?curid=${props.result.pageid}`;
    });

    return { url };
  },
};
</script>