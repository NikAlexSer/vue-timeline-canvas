<template>
  <div class="timeline-container">
    <canvas id="timeline" width="1500" height="62" class="timeline-canvas" ondragstart="return false;"></canvas>
    <button @click="startTime">Lol</button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
  name: 'Timeline',
  data: function () {
    return {
      canvas: null,
      ctx: null,
      canvasW: null,
      canvasH: null,
      minutes_per_step: [0.1, 1, 2, 5, 10, 15, 20, 30, 60, 120, 180, 240, 360, 720, 1440, 2880],
      graduation_step: 20,
      hours_per_ruler: 24,
      start_timestamp: new Date().getTime() - 12*60*60*1000,
      distance_between_gtitle: 80,
      zoom: 24,
      g_isMousedown: false,
      g_isMousemove: false, 
      g_mousedownCursor: null,
      returnTime: null,

      timecell: new Array
    }
  },
  props: {
   
  },
  mounted() {
    for (let i = 1; i < 1000; i++) {
      this.timecell.push({
        "beginTime": new Date().getTime() + i * 30000,
        "endTime": new Date().getTime() + i * 30000 - 10000,
        "style": {
          "background":"rgba(132, 244, 180, 1)"
        }
      })
    }

    console.log(this.timecell)

    this.canvas = document.getElementById("timeline")
    this.ctx = this.canvas.getContext("2d");
    this.canvasW = this.canvas.width;
    this.canvasH = this.canvas.height;
    this.init()
  },
  methods: {
    init: function() {
      this.drawCellBg();
      this.add_graduations(this.start_timestamp);
      this.drawLine(0, this.canvasH, this.canvasW, this.canvasH, "rgb(151, 158, 167)", 1); 
      this.drawLine(this.canvasW/2, 0, this.canvasW/2, 33, "rgb(64, 196, 255", 2); 
      this.add_events()
      this.add_cells(this.timecell)
      
      let time = this.start_timestamp + (this.hours_per_ruler*3600*1000)/2;
      this.ctx.fillStyle = "rgb(64, 196, 255";
      this.ctx.fillText(this.changeTime(time), this.canvasW/2-60, 50);
    },
    add_events: function() {
      this.canvas.addEventListener('mousewheel', this.mousewheelFunc);
      this.canvas.addEventListener('mousedown', this.mousedownFunc);
      this.canvas.addEventListener('mousemove', this.mousemoveFunc);
      this.canvas.addEventListener('mouseup', this.mouseupFunc);
      this.canvas.addEventListener('mouseout', this.mouseoutFunc);
    },
    add_cells: function(cells) {
      cells.forEach(cell => {
        this.draw_cell(cell)
      });
    },
    draw_cell: function(cell) {
      let px_per_ms = this.canvasW / (this.hours_per_ruler * 60 * 60 * 1000); // px/ms
      var beginX = (cell.beginTime - this.start_timestamp) * px_per_ms;
      var cell_width = (cell.beginTime - cell.endTime) * px_per_ms;
      this.ctx.fillStyle = cell.style.background;
      this.ctx.fillRect(beginX,0,cell_width,15);
    },
    drawCellBg: function() {
      this.ctx.fillStyle = "rgba(69, 72, 76, 0.5)";
      this.ctx.fillRect(0, 0, this.canvasW, 15); 
    },
    drawLine: function(beginX, beginY, endX, endY, color, width) {
      this.ctx.beginPath();
      this.ctx.moveTo(beginX,beginY);
      this.ctx.lineTo(endX,endY);
      this.ctx.strokeStyle = color;
      this.ctx.lineWidth = width;
      this.ctx.stroke();
    },
    ms_to_next_step: function(timestamp, step) {
      let remainder = timestamp % step;
      return remainder ? step - remainder : 0;
    },
    graduation_title: function(datetime) {
      if (datetime.getHours() == 0 && datetime.getMinutes() == 0 && datetime.getMilliseconds() == 0) {
        return ('0' + datetime.getDate().toString()).substr(-2) + '.' +
              ('0' + (datetime.getMonth() + 1).toString()).substr(-2) + '.' +
              datetime.getFullYear();
      }
        return datetime.getHours() + ':' + ('0' + datetime.getMinutes().toString()).substr(-2);
    },
    add_graduations: function(start_timestamp) {
      let 
        px_per_min = this.canvasW / (this.hours_per_ruler * 60), // px/min
        px_per_ms = this.canvasW / (this.hours_per_ruler * 60 * 60 * 1000), // px/ms
        px_per_step = this.graduation_step,
        min_per_step = px_per_step / px_per_min; // min

      for (let i = 0; i < this.minutes_per_step.length; i++) { 
          if (min_per_step <= this.minutes_per_step[i]) { 
              min_per_step = this.minutes_per_step[i]; 
              px_per_step = px_per_min * min_per_step;
              break
          }
      }

      let medium_step = 30;

      for (let i = 0; i < this.minutes_per_step.length; i++) {
          if (this.distance_between_gtitle / px_per_min <= this.minutes_per_step[i]) {
              medium_step = this.minutes_per_step[i];
              break;
          }
      }

      let 
        num_steps = this.canvasW / px_per_step,
        graduation_left,
        graduation_time,
        caret_class,
        lineH,
        ms_offset = this.ms_to_next_step(start_timestamp,min_per_step*60*1000),
        px_offset = ms_offset * px_per_ms,
        ms_per_step = px_per_step / px_per_ms;

      for (let i = 0; i < num_steps; i++){
        graduation_left = px_offset + i * px_per_step; 
        graduation_time = start_timestamp + ms_offset + i * ms_per_step; 
        let date = new Date(graduation_time);

        if (date.getUTCHours() == 0 && date.getUTCMinutes() == 0) {
          caret_class = 'big';
          lineH = 25;
          let big_date = this.graduation_title(date);
          this.ctx.fillText(big_date, graduation_left-20, 30);
          this.ctx.fillStyle = "rgba(151,158,167,1)";
        } else if (graduation_time / (60 * 1000) % medium_step == 0) {
          caret_class = 'middle';
          lineH = 15;
          let middle_date = this.graduation_title(date);
          this.ctx.fillText(middle_date, graduation_left-20, 30);
          this.ctx.fillStyle = "rgba(151,158,167,1)";
        } else{
          lineH = 10;
        }
        this.drawLine(graduation_left,0,graduation_left,lineH,"rgba(151,158,167,1)",1);
      }
    },
    get_cursor_x_position: function(e) {
      let posx = 0;

      if (!e) {
        e = window.event;
      }
      
      if (e.pageX || e.pageY) {
        posx = e.pageX;
      } else if (e.clientX || e.clientY) {
        posx = e.clientX + document.body.scrollLeft + document.documentElement.scrollLeft;
      }
      
      return posx;
    },
    mousewheelFunc: function() {
      if (event && event.preventDefault){
        event.preventDefault()
      } else {
          window.event.returnValue = false;
          return false;
      }	

      var e = window.event || event;
      var delta = Math.max(-1,Math.min(1,(e.wheelDelta || -e.detail)));
      var middle_time = this.start_timestamp + (this.hours_per_ruler*3600*1000)/2; //ms
      if (delta < 0) {
          this.zoom = this.zoom + 4;
      if (this.zoom >= 24) {
        
          }
          this.hours_per_ruler = this.zoom;
      } else if (delta > 0) {
          this.zoom = this.zoom - 4;
      if (this.zoom <= 1) {
        this.zoom = 0.1;
          }
          this.hours_per_ruler = this.zoom;
      }	

      this.clearCanvas();
      this.start_timestamp = middle_time - (this.hours_per_ruler*3600*1000)/2; 
      this.init()
    },
    mousedownFunc: function(e) {
      this.g_isMousedown = true;
      this.g_mousedownCursor = this.get_cursor_x_position(e);
    },
    mousemoveFunc: function(e) {
      let pos_x = this.get_cursor_x_position(e)
      let px_per_ms = this.canvasW / (this.hours_per_ruler * 60 * 60 * 1000); // px/ms
      this.clearCanvas();
      if (this.g_isMousedown) {
          let diff_x = pos_x - this.g_mousedownCursor;
          this.start_timestamp = this.start_timestamp - Math.round(diff_x / px_per_ms);
          this.init();
          this.g_isMousemove = true;
          this.g_mousedownCursor = pos_x;
      } else {
          let time = this.start_timestamp + pos_x/px_per_ms;
          this.init();
          this.drawLine(pos_x,0,pos_x,50,"rgb(194, 202, 215)",1);
          this.ctx.fillStyle = "rgb(194, 202, 215)";
          this.ctx.fillText(this.changeTime(time),pos_x-50,60);
      }
    },
    mouseupFunc: function(e) {
      if (this.g_isMousemove) { 
        this.g_isMousemove = false;
        this.g_isMousedown = false;
        this.returnTime = this.start_timestamp + (this.hours_per_ruler*3600*1000)/2;
      } else{ // click 
        this.g_isMousedown = false;
        var posx = this.get_cursor_x_position(e); 
        var ms_per_px = (this.zoom * 3600 * 1000) / this.canvasW; // ms/px
        this.returnTime = this.start_timestamp + posx * ms_per_px;
        this.set_time_to_middle(this.returnTime);
    }
    },
    mouseoutFunc: function() {
      this.clearCanvas();
      this.init();
    },
    
    clearCanvas: function() {
      this.ctx.clearRect(0,0,1500,150);
    },
    changeTime: function(time) {
      let newTime = new Date(time),
          year = newTime.getFullYear(),
          month = (newTime.getMonth() + 1) < 10 ? '0' + (newTime.getMonth() + 1) : (newTime.getMonth() + 1),
          date = newTime.getDate() < 10 ? '0' + newTime.getDate() : newTime.getDate(),
          hour = newTime.getHours() < 10 ? '0' + newTime.getHours() : newTime.getHours(),
          minute = newTime.getMinutes() < 10 ? '0' + newTime.getMinutes() : newTime.getMinutes(),
          second = newTime.getSeconds() < 10 ? '0' + newTime.getSeconds() : newTime.getSeconds();

      return year + "-" + month + "-" + date + " " + hour + ":" + minute + ":" + second;
    },
    set_time_to_middle: function(time) {
      this.clearCanvas();
      this.start_timestamp = time - (this.hours_per_ruler * 60 * 60 * 1000)/2;
      this.init()
    },
    startTime: function() {
      let self = this
      let timeMoving = setInterval(function(){
        self.set_time_to_middle(new Date().getTime());
      },100)
    }
  }
});
</script>


<style scoped lang="scss">
.timeline-container {
  margin: 0;
  padding: 0;
}
.timeline-canvas {
  cursor: pointer;
  border:1px solid #2b2f33;
  background-color: #2b2f33;
}
</style>
