<template>
  <div id="app">
    <h1>Jones Polynomial</h1>
    <DrawManager ref="draw_manager"/>
    <button @click="submit()" v-if="configured">submit</button>
  </div>
</template>

<script>
import DrawManager from './components/DrawManager.vue'
export default {
  name: 'app',
  data: function(){
    return{
      configured: false
    }
  },
  methods: {
    submit: function(){
      //送信
      alert(this.$refs.draw_manager.dump_link_json())
    }
  },
  mounted: function(){
    this.$watch(
      "$refs.draw_manager.state",
    function(){
      this.configured=(this.$refs.draw_manager.state == 'Configured');
    })
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
