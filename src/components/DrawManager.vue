<template>
  <div @mouseup="suspend()" @mousedown="resume()">
    <Canvas ref="canvas"/>
    <h3 v-if="state=='Done'">You've made a link diagram! congrats!</h3>
    <button @click="configure_link()" v-if="state=='Done'">configure a link!</button>
    <h3 v-if="state=='Configured'">You can submit this link and get the Jones polynomial!</h3>
  </div>
</template>

<script>
// Canvasの書き込み制御、データをいい感じで計算するコンポーネント
import Canvas from './Canvas.vue'
export default {
  name: 'draw_manager',
  props: {
    minimum_radius_of_loop: {
      type: Number,
      default: 30
    },
    minimum_distance_for_resume: {
      type: Number,
      default: 30
    },
    erase_radius: {
      type: Number,
      default: 15
    },
    divide_x: {
      type: Number,
      default: 10
    },
    divide_y: {
      type: Number,
      default: 10
    }
  },
  data: function() {
    return {
      //代数的データ型が欲しいけど、諦める
      // state = Ready | Draw | Suspend | Done | Configured
      // Draw  = DrawUnTerminable | DrawTerminable
      state: 'Ready',
      state_sub: 'DrawUnTerminable',
      initial_place: null,
      intersection_pairs:      [],
      trajectory:              []
    }
  },
  components: {
    Canvas
  },
  methods: {
    dump_link_json(){
      var intersection_edge_list = []
      this.intersection_pairs.forEach(function(p){
        Array.prototype.push.apply(intersection_edge_list,p);
      })
      intersection_edge_list.sort(function(a,b){return a-b;});
      var edge_max_index = intersection_edge_list.length;
      var ret_json = {};
      this.intersection_pairs.forEach(function(p,ind){
        var back_ind  = intersection_edge_list.indexOf(p[0]);
        var front_ind = intersection_edge_list.indexOf(p[1]);
        ret_json[ind] = [front_ind,(front_ind+1)%edge_max_index,back_ind,(back_ind+1)%edge_max_index];
      });
      return JSON.stringify(ret_json)
    },
    distance(a2d,b2d){
      return Math.sqrt((a2d[0]-b2d[0])*(a2d[0]-b2d[0])+(a2d[1]-b2d[1])*(a2d[1]-b2d[1]))
    },
    resume(){
      var c = this.$refs.canvas;
      if(this.state == 'Suspend' && this.distance([c.current_x,c.current_y],[c.last_draw_x,c.last_draw_y]) < this.minimum_distance_for_resume){
        this.$refs.canvas.draw_enable = true
        this.state='Draw';
      }
    },
    suspend(){
      if(!(this.state=='Done' || this.state=='Configured')){
        this.state='Suspend';
        this.$refs.canvas.draw_enable = false;
      }
    },
    //2つのLineが交差するかどうか
    is_intersect(idx,idy){
      var l1_from  = this.trajectory[idx];
      var l1_to    = this.trajectory[idx+1];
      var l2_from  = this.trajectory[idy];
      var l2_to    = this.trajectory[idy+1];
      var line_formula = function(l){
        return l[1]-l1_from[1]-(l[0]-l1_from[0])*(l1_to[1]-l1_from[1])/(l1_to[0]-l1_from[0]);
      }
      return line_formula(l2_from)*line_formula(l2_to)<0;
    },
    intersection_position(idx,idy){
      var l1_from  = this.trajectory[idx];
      var l1_to    = this.trajectory[idx+1];
      var l2_from  = this.trajectory[idy];
      var l2_to    = this.trajectory[idy+1];
      var retx = (l1_from[1]-l2_from[1]
                  -l1_from[0]*(l1_to[1]-l1_from[1])/(l1_to[0]-l1_from[0])
                  +l2_from[0]*(l2_to[1]-l2_from[1])/(l2_to[0]-l2_from[0]))
                  /((l2_to[1]-l2_from[1])/(l2_to[0]-l2_from[0])-(l1_to[1]-l1_from[1])/(l1_to[0]-l1_from[0]));
      return [retx,l1_from[1]+retx*(l1_to[1]-l1_from[1])/(l1_to[0]-l1_from[0])-l1_from[0]*(l1_to[1]-l1_from[1])/(l1_to[0]-l1_from[0])]
    },
    rearrange_projection_diagram(){
      this.intersection_pairs.forEach(function(pair){
        /*
        this.$refs.canvas.ctx.beginPath();
        this.$refs.canvas.ctx.moveTo(back_from[0], back_from[1]);
        this.$refs.canvas.ctx.lineTo(back_to[0], back_to[1]);
        this.$refs.canvas.ctx.clip();
        */
        var intersection_pos = this.intersection_position(pair[0],pair[1]);
        this.$refs.canvas.ctx.clearRect(intersection_pos[0]-this.erase_radius/2,
                                        intersection_pos[1]-this.erase_radius/2,
                                        this.erase_radius,
                                        this.erase_radius);

        [...Array(5).keys()].forEach(function(ind){
          var front_from = this.trajectory[Math.min(pair[1]+ind,this.trajectory.length-2)];
          var front_to   = this.trajectory[Math.min(pair[1]+ind,this.trajectory.length-2)+1];
          this.$refs.canvas.ctx.beginPath();
          this.$refs.canvas.ctx.moveTo(front_from[0], front_from[1]);
          this.$refs.canvas.ctx.lineTo(front_to[0], front_to[1]);
          this.$refs.canvas.ctx.stroke();

          front_from = this.trajectory[Math.max(0,pair[1]-ind)];
          front_to   = this.trajectory[Math.max(0,pair[1]-ind)+1];
          this.$refs.canvas.ctx.beginPath();
          this.$refs.canvas.ctx.moveTo(front_from[0], front_from[1]);
          this.$refs.canvas.ctx.lineTo(front_to[0], front_to[1]);
          this.$refs.canvas.ctx.stroke();
        },this)
      },this)
      this.state = 'Configured';
    },
    configure_link(){
      var trajectory_len = this.trajectory.length;
      // global search O(trajectory_len^2)
      for(var idx=0;idx<trajectory_len-1;idx++){
        for(var idy=idx+2;idy<trajectory_len-1;idy++){
          if(this.is_intersect(idx,idy) && this.is_intersect(idy,idx)){
            this.intersection_pairs.push([idx,idy])
          }
        }
      }
      /*
      var cluster_index = this.cluster();
      Array(this.divide_x).forEach(function (idx){
        Array(this.divide_y).forEach(function (idy){
          intersection_pairs += this.detect_intersection_in_a_cluster(idx,idy);
        })
      })
      */
      this.rearrange_projection_diagram();
    }
  },
  mounted(){
    this.$watch(
      "$refs.canvas.last_draw_x",
      function(){
        var newx = this.$refs.canvas.last_draw_x;
        var newy = this.$refs.canvas.last_draw_y;
        this.trajectory.push([newx,newy]);
        //stationary processing
        if(this.state == 'Ready'){
          this.state = 'Draw';
          this.state_sub = 'DrawUnTerminable';
          this.initial_place = [newx,newy];
        }else if(this.state == 'Draw' && this.distance(this.initial_place,[newx,newy])>this.minimum_radius_of_loop){
          this.state_sub = 'DrawTerminable';
        }else if(this.state_sub == 'DrawTerminable' && this.distance(this.initial_place,[newx,newy])<this.minimum_radius_of_loop){
          this.state = 'Done';
          this.$refs.canvas.draw_core(newx,newy,this.initial_place[0],this.initial_place[1]);
          this.$refs.canvas.draw_enable = false;
        }
      }
    );
  }
}
</script>
