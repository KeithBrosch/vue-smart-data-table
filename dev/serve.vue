<script>
import Vue from 'vue';
import VueSmartDataTable from '@/vue-smart-data-table.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueSmartDataTable
  },
  data() {
    return {
      users: [
        {
          'First Name': "Apple",
          'Last Name': "Joose",
          'Points': 6,
        },
        {
          'First Name': "Merk",
          'Last Name': "Kifflefom",
          'Points': 0,
        },
        {
          'First Name': "Clay",
          'Last Name': "Greggle",
          'Points': 0,
        },
      ],
      actions: [
        'Edit',
        'Duplicate',
        'anotherAction',
      ],
    };
  },
});
</script>

<template>
  <div id="app">
    <vue-smart-data-table :data="users" :actions="actions" />
  </div>
</template>
