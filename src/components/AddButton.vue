<template>
  <button
    type="button"
    class="mt-3 w-32 inline-flex items-center justify-center py-3 px-6 shadow-sm text-sm font-medium rounded-md text-white bg-purple-900"
  >
    Добавить
  </button>
</template>
<script>
export default {
  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false
    }
  }
};
</script>
