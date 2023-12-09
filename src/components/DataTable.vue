<template>
  <table class="m-2 text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
    <caption
      class="p-5 text-lg font-semibold text-left rtl:text-right text-gray-900 bg-white dark:text-white dark:bg-gray-800"
    >
      {{
        caption
      }}
    </caption>
    <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
      <tr>
        <th v-for="header in headers" :key="header" scope="col" class="px-6 py-3">{{ header }}</th>
      </tr>
    </thead>
    <tbody>
      <slot></slot>
    </tbody>
  </table>
</template>

<script>
export default {
  props: {
    caption: {
      type: String,
      required: true
    },
    headers: {
      type: Array,
      required: true
    }
  }
}
</script>
