<template>
  <section class="bg-white px-4 pt-4 rounded-lg">
    <div>
      <div class="wallet-block border-gray-300 rounded-md">
        <div
          class="flex flex-col h-16 bg-white mt-1 relative shadow-md pr-10 border-gray-300 text-gray-900  rounded-md"
        >
          <label
            for="wallet"
            class="text-xs pl-2 pt-2 font-medium text-gray-700"
          >
            Тикер
          </label>
          <input
            v-model="ticker"
            @keydown.enter="add"
            type="text"
            name="wallet"
            id="wallet"
            autocomplete="off"
            class="border-none rounded-md focus:outline-none focus:ring-0 focus:border-0"
            placeholder="Например DOGE"
          />
        </div>
      </div>
    </div>
    <add-button @click="add" type="button" class="my-4" />
  </section>
</template>

<script>
import AddButton from "./AddButton.vue";

export default {
  components: {
    AddButton
  },

  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false
    },
    error: {
      type: Boolean,
      required: false,
      default: false
    }
  },

  emits: {
    "add-ticker": value => typeof value === "string" && value.length > 0,
    "ticker-input": value => typeof value === "string"
  },

  data() {
    return { ticker: "" };
  },

  methods: {
    add() {
      if (this.ticker.length === 0) {
        return;
      }
      this.$emit("add-ticker", this.ticker);
      this.ticker = "";
    }
  }
};
</script>
<style lang="postcss">
.wallet-block {
  max-width: 440px;
}
</style>
