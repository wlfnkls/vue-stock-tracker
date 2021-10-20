<template>
  <div class="max-w-2xl mx-auto mt-8">
    <form
      v-on:submit.prevent="fetchStockData"
      @keyup="search"
      class="
        rounded-xl
        shadow-xl
        bg-white
        flex
        justify-between
        align-middle
        p-2
      "
    >
      <input
        type="text"
        name="stock-symbol"
        id="stock-symbol"
        v-model="stockSymbol"
        class="rounded-lg pl-2 uppercase outline-none w-full"
        placeholder="Symbol (AAPL, ...)"
      />
      <input
        type="button"
        value="search"
        class="
          bg-green-400
          rounded-lg
          uppercase
          text-white
          font-bold
          p-3
          cursor-pointer
          ml-2
        "
        @click="fetchStockData"
      />
    </form>
    <ul
      v-if="searchSuggestions && searchSuggestions.length"
      class="divide-y-2 divide-gray-100 bg-white rounded-xl p-2 mt-4 shadow-xl"
    >
      <li
        v-for="suggestion in searchSuggestions"
        :key="suggestion['1. symbol']"
        class="p-2 cursor-pointer"
        @click="selectSuggestion(suggestion['1. symbol'])"
      >
        {{ suggestion["1. symbol"] }} - {{ suggestion["2. name"] }}
      </li>
    </ul>
  </div>
  <stock-summary
    v-if="Object.keys(stockData).length && Object.keys(stockOverview).length"
    :stockData="stockData"
    :stockOverview="stockOverview"
  />
</template>

<script>
import axios from "axios";
import StockSummary from "./StockSummary.vue";
export default {
  components: {
    StockSummary,
  },
  data() {
    return {
      timeout: null,
      stockSymbol: "",
      searchSuggestions: [],
      stockData: {},
      stockOverview: {},
      apiKey: import.meta.env.VITE_API_KEY,
    };
  },
  methods: {
    async fetchStockData() {
      try {
        [this.stockData, this.stockOverview] = await axios.all([
          this.fetchGlobalQuote(),
          this.fetchOverview(),
        ]);
        this.searchSuggestions = [];
        this.stockSymbol = '';
      } catch (err) {
        console.log(err);
      }
    },
    async fetchGlobalQuote() {
      try {
        const stockData = await axios.get(
          `https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=${this.stockSymbol}&interval=5min&apikey=${this.apiKey}`
        );
        return stockData.data;
      } catch (err) {
        console.log(err);
      }
    },
    async fetchOverview() {
      try {
        const stockData = await axios.get(
          `https://www.alphavantage.co/query?function=OVERVIEW&symbol=${this.stockSymbol}&interval=5min&apikey=${this.apiKey}`
        );
        return stockData.data;
      } catch (err) {
        console.log(err);
      }
    },
    async autoComplete() {
      try {
        const suggestions = await axios.get(
          `https://www.alphavantage.co/query?function=SYMBOL_SEARCH&keywords=${this.stockSymbol}&apikey=${this.apiKey}`
        );
        this.searchSuggestions = suggestions.data.bestMatches;
      } catch (error) {
        console.log(error);
      }
    },
    search() {
      clearTimeout(this.timeout);

      this.timeout = setTimeout(() => {
        this.autoComplete();
      }, 1000);
    },
    selectSuggestion(symbol) {
      this.stockSymbol = symbol;
      this.searchSuggestions = [];
      this.fetchStockData();
    },
  },
};
</script>

<style>
</style>