<template>
  <div :class="$options.name"></div>
</template>

<script>
/**
* @file Event1.vue
* @author shenjp@founder.com
* @date 2020-09-11 13:33:50
*/
export default {
  name: 'Event1',
  data () {
    return {
    };
  },
  mounted () {
    this.$on('event1', () => {
      console.log('ok');
    });

    this.$on(['event1', 'event2', 'event3'], () => {
      console.log('ok');
    });

    setTimeout(() => {
      this.$emit('event1');
    }, 1000);

    // this.$off();
    // this.$off('event2');

    this.$root.$on('my-event', () => {
      console.log('root event ok');
    });
  }
};
</script>

<style lang="scss" scoped>

</style>
