<template>
  <div id="app">
    <h1>Kauffman Bracket</h1>
    <DrawManager ref="draw_manager"/>
    <button @click="submit()" v-if="configured && kauffman_bracket==null">submit</button>
    <h2 v-if="kauffman_bracket" id="kauffman_bracket">The Kauffman bracket of this link is \({{kauffman_bracket}}\)</h2>
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
      kauffman_bracket: null,
      url: '/jones/post'
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
        this.kauffman_bracket = res.data
        this.$nextTick(()=>MathJax.Hub.Queue(["Typeset",MathJax.Hub,"kauffman_bracket"]))
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
