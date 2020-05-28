<template>
  <h3>Всего задач: {{finished + unfinished}} (сделано {{finished}}, в работе {{unfinished}})</h3>
</template>

<style>
  h3 {
    text-align: left;
  }
</style>

<script>
export default {
  props: {
    finished: Number,
    unfinished: Number,
  },
};
</script>
