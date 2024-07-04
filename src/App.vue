<template>
  <div id="app">
    <DynamicTable />
  </div>
</template>

<script>
import DynamicTable from './assets/components/DynamicTable.vue';

export default {
  components: {
    DynamicTable
  }
};

</script>

<style>
  .circular{
    width: 90px;
    height: 90px;
    background-size: cover;
    border-radius: 50px;
    -webkit-border-radius: 50px;
    -moz-border-radius: 50px;
  }
</style>
