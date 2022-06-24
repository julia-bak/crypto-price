<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div class="container">
      <div class="w-full my-4"></div>
      <add-ticker
        @add-ticker="add"
        :disabled="tooManyTickersAdded"
        :error="error"
      />
      <template v-if="tickers.length">
        <div
          class="filter-block h-12 mt-3 d-f rounded-md shadow-md border-gray-300 bg-white"
        >
          <label for="filter" class="flex items-center relative">
            <svg
              width="15"
              height="15"
              viewBox="0 0 18 18"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
              class="pointer-events-none w-15 h-15 absolute top-1/2 transform -translate-y-1/2 left-3"
            >
              <path
                d="M16.5 16.5L11.5 11.5M13.1667 7.33333C13.1667 10.555 10.555 13.1667 7.33333 13.1667C4.11167 13.1667 1.5 10.555 1.5 7.33333C1.5 4.11167 4.11167 1.5 7.33333 1.5C10.555 1.5 13.1667 4.11167 13.1667 7.33333Z"
                stroke="#9CA3AF"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            <input
              v-model="filter"
              @input="page = 1"
              @keydown.enter="add"
              type="text"
              id="filter"
              name="filter"
              class="ml-5  text-gray-900 border-none focus:outline-none focus:ring-0 focus:border-0 rounded-md"
              placeholder="Найти тикер"
            />
          </label>
        </div>
        <dl class="mt-5 grid grid-cols-1 gap-3 sm:grid-cols-3">
          <div
            v-for="t in paginatedTickers"
            :key="t.name"
            @click="select(t)"
            :class="{
              'border-4': selectedTicker === t
            }"
            class="flex justify-between bg-white h-24 overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
          >
            <div class="p-4 text-left">
              <dt class="text-sm font-medium text-gray-900 truncate">
                {{ t.name }}
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ formatPrice(t.price) }}$
              </dd>
            </div>

            <div @click.stop="handleDelete(t)" class="flex items-top p-4">
              <svg
                width="16"
                height="18"
                viewBox="0 0 16 18"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path d="M13.8335 4.83333L13.1107 14.9521C13.0484 15.8243 12.3227 16.5 11.4483 16.5H4.55203C3.67763 16.5 2.9519 15.8243 2.8896 14.9521L2.16683 4.83333M6.3335 8.16667V13.1667M9.66683 8.16667V13.1667M10.5002 4.83333V2.33333C10.5002 1.8731 10.1271 1.5 9.66683 1.5H6.3335C5.87326 1.5 5.50016 1.8731 5.50016 2.33333V4.83333M1.3335 4.83333H14.6668" stroke="#9CA3AF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
          </div>
        </dl>
      </template>
      <section v-if="selectedTicker" class="relative">
        <div
          class="mt-4 flex items-end border-gray-600 rounded-md bg-white h-64 p-4"
          ref="graph"
        >
          <div
            v-for="(bar, idx) in normalizedGraph"
            :key="idx"
            :style="{ height: `${bar}%` }"
            class="bg-yellow-300 border w-10"
          ></div>
        </div>
      </section>
      <div class="flex justify-between mt-4">
        <div class="flex items-center text-gray-700 text-sm">
          Показано {{ paginatedTickers.length }} результатов из
          {{ tickers.length }}
        </div>
        <div class="flex justify-end">
          <button
            class="my-4 inline-flex items-center py-2 px-4 border border-gray-300 shadow-sm text-sm text-gray-700  font-medium rounded-md bg-white"
            v-if="page > 1"
            @click="page = page - 1"
          >
            Назад
          </button>
          <button
            class="my-4 inline-flex items-center py-2 px-4 border border-gray-300 shadow-sm text-sm text-gray-700  font-medium rounded-md bg-white"
            @click="page = page + 1"
            v-if="hasNextPage"
          >
            Вперед
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { subscribeToTicker, unsubscribeFromTicker } from "./api";
import AddTicker from "./components/AddTicker.vue";

export default {
  name: "App",

  components: {
    AddTicker
  },

  data() {
    return {
      filter: "",

      tickers: [],
      selectedTicker: null,

      graph: [],
      maxGraphElements: 1,

      page: 1,

      error: false
    };
  },

  created() {
    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );

    if (windowData.filter) this.filter = windowData.filter;

    if (windowData.page) this.page = windowData.page;

    this.setTickerList();

    const tickersData = localStorage.getItem("cryptprice-list");
    if (tickersData) {
      this.tickers = JSON.parse(tickersData);

      this.tickers.forEach(ticker => {
        subscribeToTicker(ticker.name, (newPrice, isExist) =>
          this.updateTicker(ticker.name, newPrice, isExist)
        );
      });
    }
  },

  mounted() {
    window.addEventListener("resize", this.calculateMaxGraphElements);
  },

  beforeUnmount() {
    window.removeEventListener("resize", this.calculateMaxGraphElements);
  },

  computed: {
    tooManyTickersAdded() {
      return this.tickers.length > 4;
    },

    startIndex() {
      return (this.page - 1) * 6;
    },

    endIndex() {
      return this.page * 6;
    },

    filteredTickers() {
      return this.tickers.filter(ticker => ticker.name.includes(this.filter));
    },

    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },

    hasNextPage() {
      return this.filteredTickers.length > this.endIndex;
    },

    normalizedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);

      if (maxValue === minValue) {
        return this.graph.map(() => 50);
      }

      return this.graph.map(
        price => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
    },

    pageStateOptions() {
      return {
        filter: this.filter,
        page: this.page
      };
    }
  },

  methods: {
    calculateMaxGraphElements() {
      if (!this.$refs.graph) {
        return;
      }

      this.maxGraphElements = this.$refs.graph.clientWidth / 38;
    },

    updateTicker(tickerName, price) {
      this.tickers
        .filter(t => t.name === tickerName)
        .forEach(t => {
          if (t === this.selectedTicker) {
            this.graph.push(price);
            while (this.graph.length > this.maxGraphElements) {
              this.graph.shift();
            }
          }
          t.price = price;
        });
    },

    formatPrice(price) {
      if (price === "-") {
        return price;
      }
      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },

    setTickerList() {
      fetch(`https://min-api.cryptocompare.com/data/all/coinlist?summary=true`)
        .then(response => response.json())
        .then(response => {
          this.tickerList = Object.keys(response.Data);
        });
    },

    add(ticker) {
      if (this.tickers.find(t => t.name === ticker.toUpperCase())) {
        this.error = true;
        return;
      }

      if (this.tickerList.includes(ticker)) {
        this.filter = "";

        const newTicker = {
          name: ticker.toUpperCase(),
          price: "-"
        };

        this.tickers = [...this.tickers, newTicker];

        subscribeToTicker(newTicker.name, (newPrice, isExist) =>
          this.updateTicker(newTicker.name, newPrice, isExist)
        );

        this.error = false;
      }
    },

    select(ticker) {
      this.selectedTicker = ticker;
    },

    handleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter(t => t !== tickerToRemove);
      if (this.selectedTicker === tickerToRemove) {
        this.selectedTicker = null;
      }
      unsubscribeFromTicker(tickerToRemove.name);
    }
  },

  watch: {
    selectedTicker() {
      this.graph = [];

      this.$nextTick().then(this.calculateMaxGraphElements);
    },

    tickers(newValue, oldValue) {
      console.log(newValue === oldValue);
      localStorage.setItem("cryptoprice-list", JSON.stringify(this.tickers));
    },

    paginatedTickers() {
      if (this.paginatedTickers.length === 0 && this.page > 1) {
        this.page -= 1;
      }
    },

    filter() {
      this.page = 1;
    },

    pageStateOptions(value) {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${value.filter}&page=${value.page}`
      );
    }
  }
};
</script>

<style lang="postcss" scoped>
.filter-block {
  max-width: 456px;
}
</style>
