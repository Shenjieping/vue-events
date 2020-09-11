<template>
  <div id="app">
    <h1>event1</h1>
    <Event1/>
    
    <h1>event2</h1>
    <Event2 @hook:updated="updateCounter"/>
  </div>
</template>

<script>
import Event1 from './components/Event1.vue'
import Event2 from './components/Event2.vue'

export default {
  name: 'App',
  components: {
    Event1,
    Event2
  },
  methods: {
    updateCounter () {
      console.log('hooks ok');
    }
  }
}
</script>

