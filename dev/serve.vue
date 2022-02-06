<script>
import Vue from 'vue';
import VueSmartDataTable from '@/vue-smart-data-table.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueSmartDataTable
  }
});
</script>

<template>
  <div id="app">
    <vue-smart-data-table />
  </div>
</template>
