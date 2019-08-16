<template>
  <div id="app">
    <h1>Jones Polynomial</h1>
    <DrawManager ref="draw_manager"/>
    <button @click="submit()" v-if="configured && jones_polynomial==null">submit</button>
    <h2 v-if="jones_polynomial" id="jones_polynomial">The Jones polynomial of this link is \({{jones_polynomial}}\)</h2>
  </div>
</template>

<script>
import Vue from 'vue'
import LoadScript from 'vue-plugin-load-script';
Vue.use(LoadScript);
Vue.loadScript("https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML")
  .then(()=>{MathJax.Hub.Config({messageStyle: "none"})})
import DrawManager from './components/DrawManager.vue'
import axios from 'axios'

export default {
  name: 'app',
  data: function(){
    return{
      configured: false,
      jones_polynomial: null,
      url: 'http://localhost:8080/jones/post'
    }
  },
  methods: {
    submit: function(){
      var config = {
        headers:{
          'Content-Type':'application/json'
        },
        withCredentials:true,
      }
      axios.post(this.url,this.$refs.draw_manager.dump_link_json(),config)
      .then(function(res){
        this.jones_polynomial = res.data
        this.$nextTick(()=>MathJax.Hub.Queue(["Typeset",MathJax.Hub,"jones_polynomial"]))
      }.bind(this))
      .catch(function(res){
        alert(res)
      })
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
