<template>
  <div
    v-if="stockData['Global Quote']"
    class="
      max-w-2xl
      p-4
      rounded-xl
      shadow-lg
      transition
      transform
      hover:shadow-xl
      hover:-translate-y-0.5
      duration-150
      ease-in-out
      mx-auto
      bg-white
      cursor-pointer
      mt-8
    "
  >
    <div class="flex justify-between">
      <h2 class="font-bold text-2xl">
        {{ stockData["Global Quote"]["01. symbol"] }}
      </h2>
      <FontAwesomeIcon
        @mouseover="filled = true"
        @mouseleave="filled = false"
        :icon="favIcon"
        class="text-yellow-400 self-center"
        @click="addToFavorites"
      />
    </div>
    <h3 class="mb-3">{{ stockOverview.Name }} {{ stockOverview.AssetType }}</h3>
    <div class="pl-2" :class="{ 'border-l-4 border-green-500': isPositive }">
      <h4 class="font-bold text-xl">
        {{ price }}
      </h4>
      <span
        class="text-red-500 font-bold text-sm"
        :class="{ 'text-green-500': isPositive }"
      >
        {{ change }} ({{
          stockData["Global Quote"]["10. change percent"]
        }})</span
      >
    </div>
  </div>
  <div
    v-if="stockData['Note']"
    class="max-w-2xl p-4 rounded-xl shadow-xl mx-auto bg-white mt-8"
  >
    <p class="font-semibold">
      {{ stockData["Note"] }}
    </p>
  </div>
</template>

<script>
import { library } from "@fortawesome/fontawesome-svg-core";
import { faStar as farStar } from "@fortawesome/free-regular-svg-icons";
import { faStar as fasStar } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

library.add(farStar, fasStar);

export default {
  props: ["stockData", "stockOverview"],
  data() {
    return {
      filled: false,
    };
  },
  components: {
    FontAwesomeIcon,
  },
  computed: {
    price() {
      return this.format(this.stockData["Global Quote"]["05. price"]);
    },
    change() {
      return this.format(this.stockData["Global Quote"]["09. change"]);
    },
    isPositive() {
      return this.stockData["Global Quote"]["05. price"] > 0;
    },
    favIcon() {
      // TODO: switch fasStar and farStar if stock is saved in local storage
      return this.filled ? fasStar : farStar;
    },
  },
  methods: {
    format(value) {
      return new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
      }).format(value);
    },
    addToFavorites() {
      let favourites = localStorage.favourites ? JSON.parse(localStorage.favourites) : [];
      let entry = {};
      entry.symbol = this.stockData["Global Quote"]["01. symbol"];
      favourites.push(entry);
      localStorage.favourites = JSON.stringify(favourites);
    },
  },
};
</script>

<style>
</style>