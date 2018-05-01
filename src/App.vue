<template>
  <div id="app">
    <!-- <img src="./assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <canvas id="canvas" v-on:mousedown="handleMouseDown" v-on:mouseup="handleMouseUp" v-on:mousemove="handleMouseMove" width="800px" height="800px"></canvas>
    <ul>
      <li v-for="user in canvases" :key="user.key">{{user}}</li>
    </ul>
  </div>
</template>

<script  >
import HelloWorld from './components/HelloWorld.vue';
import firebase from 'firebase';

const environment = {
  production: false,
  firebase: {
    apiKey: 'AIzaSyB2oqcuKQTlrwuV7YHAdFAycMZaTBrpzgo',
    authDomain: 'nx-database.firebaseapp.com',
    databaseURL: 'https://nx-database.firebaseio.com',
    projectId: 'nx-database',
    storageBucket: 'nx-database.appspot.com',
    messagingSenderId: '616704118379'
  }
};

firebase.initializeApp(environment.firebase);
const db = firebase.database();

let canvasRef = db.ref('canvas');
let linesRef = db.ref('lines');
let currentMouseLocRef = db.ref('mouseCurrent');
let existingLines = [];

export default {
  name: 'app',
  firebase: {
    canvases: canvasRef,
    lines: linesRef,
    mouseCurrent: currentMouseLocRef
  },
  components: {
    HelloWorld
  },
  data: function() {
    return {
      mouse: {
        current: {
          x: 0,
          y: 0
        },
        previous: {
          x: 0,
          y: 0
        },
        down: false
      }
    };
  },
  methods: {
    draw: function(event) {
      // requestAnimationFrame(this.draw);
      if (this.mouse.down) {
        var c = document.getElementById('canvas');

        // let c = document.createElement('canvas');

        var ctx = c.getContext('2d');

        // ctx.clearRect(0, 0, 800, 800);

        ctx.lineTo(event.pageX, event.pageY);
        ctx.strokeStyle = '#F63E02';
        ctx.lineWidth = 2;
        ctx.stroke();
        existingLines.push({ x: event.pageX, y: event.pageY });
        // this.$firebaseRefs.canvases.child(canvas['.key']).set(c.toDataURL());
        this.$firebaseRefs.lines.push({
          x: event.pageX,
          y: event.pageY
        });
      }
    },
    handleMouseDown: function(event) {
      this.mouse.down = true;
      this.mouse.current = {
        x: event.pageX,
        y: event.pageY
      };
    },
    handleMouseUp: function() {
      this.mouse.down = false;
    },
    handleMouseMove: function(event) {
      const currentM = this.mouseCurrent[0];
      if (currentM) {
        currentMouseLocRef
          .child(currentM['.key'])
          .set({ x: event.pageX, y: event.pageY });
      }
      var c = document.getElementById('canvas');
      var ctx = c.getContext('2d');
      var rect = c.getBoundingClientRect();
      var item = {
        x: event.pageX - rect.left,
        y: event.pageY - rect.top
      };
      ctx.moveTo(item.x, item.y);
      this.draw(event);
    }
  },
  mounted: () => {
    currentMouseLocRef.on('child_changed', loc => {
      console.log('um');
      if (this.mouse) {
        var c = document.getElementById('canvas');
        // const dataImage = c.toDataURL();
        // this.$firebaseRefs.users.push(dataImage);
        var rect = c.getBoundingClientRect();
        var item = {
          x: loc.val().x - rect.left,
          y: loc.val().y - rect.top
        };

        this.mouse.current.x = loc.val().x;
        this.mouse.current.y = loc.val().y;
        var ctx = c.getContext('2d');
        ctx.moveTo(item.x, item.y);
      }
    });
    linesRef.on('child_added', item => {
      const res = item.val();
      if (existingLines.findIndex(r => r.x == res.x && r.y == res.y) !== -1) {
        return;
      } else {
        var c = document.getElementById('canvas');
        // let c = document.createElement('canvas');

        var ctx = c.getContext('2d');
        console.log('wow');

        // ctx.clearRect(0, 0, 800, 800);
        // if (this.mouse) {
        //   ctx.moveTo(event.pageX, event.pageY);
        // }
        // ctx.moveTo(, res.y);
        ctx.lineTo(res.x, res.y);
        ctx.strokeStyle = '#F63E02';
        ctx.lineWidth = 2;
        ctx.stroke();
        existingLines.push({ x: res.x, y: res.y });
      }
    });
  },
  updated: function() {
    // const canvas = this.canvases[0];
    // var c = document.getElementById('canvas');
    // var image = new Image();
    // image.src = canvas['.value'];
    // image.onload = () => {
    //   c.getContext('2d').drawImage(image, 0, 0);
    // };
  },
  ready: function() {
    var c = document.getElementById('canvas');
    var ctx = c.getContext('2d');
    ctx.translate(0.5, 0.5);
    ctx.imageSmoothingEnabled = false;
  }
};
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
