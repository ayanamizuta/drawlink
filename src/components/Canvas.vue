<template>
  <canvas width="400" height="400"
   @mousedown="draw_activate()"
   @mousemove="get_current_position($event)"
   @mouseup="draw_deactivate()"
   ></canvas>
</template>

<script>
export default {
  data: function(){
    return {
      last_draw_x: null,
      last_draw_y: null,
      current_x: null,
      current_y: null,
      draw_enable: true
    }
  },
  methods: {
    draw_activate(){
      document.addEventListener("mousemove", this.draw);
    },
    draw_deactivate(){
      document.removeEventListener("mousemove", this.draw);
    },
    get_current_position(event){
      var newx = event.offsetX;
      var newy = event.offsetY;
      this.current_x = newx;
      this.current_y = newy;
    },
    draw() {
      if(this.draw_enable){
        if(this.last_draw_x == null || this.last_draw_y == null){
          this.last_draw_x = this.current_x;
          this.last_draw_y = this.current_y;
        }else{
          this.draw_core(this.last_draw_x, this.last_draw_y,this.current_x,this.current_y);
          this.last_draw_x = this.current_x;
          this.last_draw_y = this.current_y;
        }
      }
    },
    draw_core(x,y,nx,ny) {
      this.ctx.beginPath();
      this.ctx.moveTo(x, y);
      this.ctx.lineTo(nx, ny);
      this.ctx.stroke();
    }
  },
  mounted() {
    this.ctx = this.$el.getContext('2d')
  }
}
</script>

<style scoped>
.canvas {
  border: 1px solid #000;
}
</style>
