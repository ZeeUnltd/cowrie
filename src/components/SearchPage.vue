<template>
  <div>
    <!-- HEADER NAV  -->
    <nav class="search-nav">
      <div class="search-bar" :class="{'no-bg' : showResultPage}">
        <div v-if="!showResultPage">
          <!-- SEARCH BAR -->
          <form action="submit" ref="searchRef" class="flex-div">
            <!-- SEARCH BUTTON -->
            <div class="search-icon input-button">
              <button type="submit" @click.prevent="submitSearch">
                <img src="../assets/search.svg" alt="Magnifier" />
              </button>
            </div>
            <!-- SEARCH INPUT -->
            <div class="search-input">
              <input
                required
                v-model="searchKeyword"
                placeholder="Search"
                @keydown="showCloseButton()"
              />
            </div>
            <!-- CLEAR SEARCH BUTTON -->
            <div class="input-button">
              <button v-if="closeButton" @click.prevent="clearSearchInput">
                <svg
                  class="BAGAv _1azRR _18QGm"
                  version="1.1"
                  viewBox="0 0 32 32"
                  width="16"
                  height="16"
                  aria-hidden="false"
                  style="fill: #93969a"
                >
                  <path
                    d="M25.33 8.55l-1.88-1.88-7.45 7.45-7.45-7.45-1.88 1.88 7.45 7.45-7.45 7.45 1.88 1.88 7.45-7.45 7.45 7.45 1.88-1.88-7.45-7.45z"
                  />
                </svg>
              </button>
            </div>
          </form>
        </div>
        <!-- RESULT SECTION -->
        <div v-else class="mt-xs">
          <span class="result-text">
            <span class="search-title">Search Results for</span>
            <span class="search-text">&nbsp;"{{ searchKeyword}}"</span>
          </span>
          <div class="back-button">
            <button @click.prevent="clearResult">Change Search</button>
          </div>
        </div>
      </div>
    </nav>
    <div v-if="loading" class="text-center">
      <img src="../assets/copper-loader.gif" alt="Loader" class="loader-gif" />
      <p>
        <em v-if="!errorMsg">Loading...</em>
        <span v-else>{{errorMsg}}</span>
      </p>
    </div>
    <!-- IMAGES -->
    <section class="picture-zone grid" v-else>
      <figure
        v-for="(image, index) in images"
        :key="`${index+'-'+image.id}`"
        class="card"
        :style="`--photo-name:${image.user.name}; --photo-location:${image.user.location}`"
      >
        <div class="filter-animate">
          <button @click="showImagewithIndex(image)" class="c-button">
            <img :src="image.urls.small" :alt="image.alt_description" />
            <figcaption class="image-description">
              <span class="bold">{{ image.user.name}}</span>
              <br />
              <span>{{ image.user.location}}</span>
            </figcaption>
          </button>
        </div>
      </figure>
    </section>
    <!-- MODAL -->
    <div v-if="showModal" class="modal">
      <div class="input-button modal-close-button">
        <button @click.prevent="closeModal">
          <svg
            class="BAGAv _1azRR _18QGm"
            version="1.1"
            viewBox="0 0 32 32"
            width="16"
            height="16"
            aria-hidden="false"
            style="fill: #fff"
          >
            <path
              d="M25.33 8.55l-1.88-1.88-7.45 7.45-7.45-7.45-1.88 1.88 7.45 7.45-7.45 7.45 1.88 1.88 7.45-7.45 7.45 7.45 1.88-1.88-7.45-7.45z"
            />
          </svg>
        </button>
      </div>
      <div class="image-wrapper">
        <div class="image-full">
          <img :src="targetImage.urls.regular" alt="Clicked Image" />
        </div>
        <div class="image-details">
          <a :href="targetImage.links.html" target="_blank">
            <span class="photographer-name filter-animate cursor">{{ targetImage.user.name}}</span>
            <br />
            <span class="photographer-location">{{ targetImage.user.location}}</span>
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import("../assets/main.css");

export default {
  name: "SearchPage",
  data() {
    return {
      closeButton: false,
      errorMsg: "",
      images: [],
      targetImage: {},
      loading: false,
      searchResult: [],
      showResultPage: false,
      showModal: false,
      searchKeyword: ""
    };
  },
  methods: {
    clearSearchInput() {
      this.searchKeyword = "";
    },
    clearResult() {
      this.clearSearchInput();
      this.getPhotos();
    },
    closeModal() {
      this.showModal = false;
    },
    getPhotos(keyword = "") {
      this.loading = true;
      let searchkey = keyword ? keyword : "african";
      let url =
        "https://api.unsplash.com/search/photos?" +
        "client_id=" +
        process.env.VUE_APP_ACCESS_KEY +
        "&per_page=10&page=1&" +
        "query=" +
        searchkey;
      window
        .fetch(url)
        .then(res => res.json())
        .then(res => {
          this.images = res.results;
          keyword
            ? (this.showResultPage = true)
            : (this.showResultPage = false);
          this.loading = false;
          if (!this.images.length) {
            console.log("empty");
            this.loading = true;
            this.errorMsg = "No result found, please try another keyword";
            this.clearSearchInput;
            return;
          }
        })
        .catch(res => {
          this.loading = true;
          this.errorMsg = "Oops! Something went wrong";
        });
    },
    showCloseButton() {
      this.searchKeyword
        ? (this.closeButton = true)
        : (this.closeButton = false);
    },
    submitSearch() {
      !this.searchKeyword ? "" : this.getPhotos(this.searchKeyword);
    },
    showImagewithIndex(image) {
      this.showModal = true;
      this.targetImage = image;
    }
  },
  mounted() {
    this.getPhotos();
  }
};
</script>
