<template>
  <div class="search-container">
    <h1>Wiki Search</h1>

    <form class="search-box" @submit.prevent="handleSearch">
      <input
        type="search"
        v-model="search"
        placeholder="Search for something..."
      />
    </form>
    <p v-if="searchInfo.totalhits">Total Results: {{ searchInfo.totalhits }}</p>

    <p v-if="showNoResult">No Results Found.</p>
  </div>

  <div class="results">
    <Result v-for="result in results" :key="result.pageid" :result="result" />
  </div>

  <a class="load-more" v-show="showLoadMore" @click.prevent="loadMoreResults">
    Load More
  </a>

  <div class="footer-credit">
    <p>
      Powered by
      <a href="http://en.wikipedia.org" target="_blank">Wikipedia</a>
    </p>
  </div>
</template>

<script>
import { ref } from "@vue/reactivity";
import Result from "./components/Result.vue";

export default {
  name: "App",
  components: { Result },
  setup() {
    const search = ref("");
    const results = ref([]);
    const currentResults = ref({});
    const searchInfo = ref({});
    const showLoadMore = ref(false);
    const showNoResult = ref(false);

    const handleSearch = async () => {
      if (search.value === "") return;

      const endpoint = `https://en.wikipedia.org/w/api.php?action=query&format=json&list=search&prop=info&inprop=url&utf8=&origin=*&srlimit=20&srsearch=${search.value}`;

      const response = await fetch(endpoint);

      if (!response.ok) {
        console.log(response.statusText);
      }

      const data = await response.json();

      results.value = data.query.search;
      currentResults.value = data.continue ?? {};
      searchInfo.value = data.query.searchinfo;

      showLoadMore.value =
        currentResults.value.sroffset &&
        searchInfo.value.totalhits > 0 &&
        currentResults.value.sroffset < searchInfo.value.totalhits
          ? true
          : false;

      showNoResult.value = results.value.length === 0 ? true : false;
    };

    const loadMoreResults = async () => {
      const endpoint = `https://en.wikipedia.org/w/api.php?action=query&format=json&list=search&prop=info|pageimages&inprop=url&utf8=&origin=*&srlimit=20&srsearch=${search.value}&continue=${currentResults.value.continue}&sroffset=${currentResults.value.sroffset}`;
      const response = await fetch(endpoint);
      if (!response.ok) {
        console.log(response.statusText);
      }

      const data = await response.json();

      currentResults.value = data.continue ?? {};

      results.value = [...results.value, ...data.query.search];

      showLoadMore.value = currentResults.value.sroffset ? true : false;
    };

    return {
      search,
      handleSearch,
      results,
      showLoadMore,
      showNoResult,
      searchInfo,
      loadMoreResults,
    };
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS", sans-serif;
}

body {
  background-color: #dddddd;
}

#app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  min-height: 100vh;
  max-width: 90vw;
  width: 100%;
  margin: 0 auto;
  padding-top: 1.5rem;
}

#app .footer-credit {
  justify-self: flex-end;
  align-self: center;
}

.search-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 36px;
}

h1 {
  color: #f5a962;
  font-size: 42px;
  text-transform: uppercase;
  text-align: center;
  margin-bottom: 16px;
}

.search-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 0px 16px 0px 16px;

  overflow: hidden;
  width: 100%;
  max-width: 480px;
  margin-bottom: 16px;
  transition: 0.4s;
}

.search-box input {
  display: block;
  appearance: none;
  outline: none;
  border: none;

  width: 100%;
  padding: 16px;
  transition: 0.4s;
}

.search-box:focus-within {
  border-radius: 16px 0px 16px 0px;
  box-shadow: 3px 3px 6px rgba(0, 0, 0, 0.2);
  transform: 0.4s;
}

.results {
  max-width: 768px;
  width: 100%;
}

.result {
  padding: 16px;
  width: 100%;
  background-color: #fff;
  border-radius: 16px;
  transform: 0.4s;
  margin-bottom: 16px;
  overflow: auto;
}

.result:hover {
  box-shadow: 3px 3px 6px rgba(0, 0, 0, 0.2);
}

.result h3 {
  color: #125d98;
  font-size: 28px;
  margin-bottom: 16px;
}

.result p {
  color: #313131;
  font-size: 18px;
  margin-bottom: 16px;
}

.result p .searchmatch {
  font-weight: 700;
  color: #3c8dad;
}

.result a {
  display: inline-block;
  color: #3c8dad;
  padding: 8px 0;
  text-decoration: underline;
  font-weight: 700;
  transition: 0.4s;
}

.result a:hover {
  color: #e8f6ef;
}

.load-more {
  cursor: pointer;
  padding: 12px;
  width: 100%;
  max-width: 768px;
  margin: 0 auto;
  border: none;
  background-color: #fff;
  color: #313131;
  border-radius: 0 0 8px 8px;
  transition: 0.4s;
  font-size: 18px;
  font-weight: 500;
  text-align: center;
}

.load-more:hover {
  transition: 0.4s;
  transform: translateY(5px);
  box-shadow: 3px 3px 6px rgba(0, 0, 0, 0.2);
}

.footer-credit {
  padding: 16px;
}

.footer-credit a {
  color: #f5a962;
  text-decoration: none;
}
</style>
