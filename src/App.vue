<template>
  <div id="app">
    <h1>Jones Polynomial</h1>
    <DrawManager ref="draw_manager"/>
    <button @click="submit()" v-if="done"></button>
  </div>
</template>

<script>
import DrawManager from './components/DrawManager.vue'
export default {
  name: 'app',
  data: function(){
    return{
      done: false
    }
  },
  methods: {
    submit: function(){
      //送信
    }
  },
  mounted: function(){
    this.done = this.$refs.draw_manager.state == 'Done'
  },
  components: {
    DrawManager
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
