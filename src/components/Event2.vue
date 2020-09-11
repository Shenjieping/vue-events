<template>
  <div :class="$options.name">
    {{ counter }}
    <button @click="handler">click me</button>
  </div>
</template>

<script>
/**
* @file Events2.vue
* @author shenjp@founder.com
* @date 2020-09-11 13:57:17
*/
export default {
  name: 'Events2',
  data () {
    return {
      counter: 0
    };
  },
  mounted () {
    setTimeout(() => {
      this.counter += 1;
    }, 2000);
  },
  updated () {
    console.log('update');
  },
  methods: {
    handler () {
      this.$root.$emit('my-event');
    }
  }
};
</script>

<style lang="scss" scoped>

</style>
