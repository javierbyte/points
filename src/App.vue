<template>
  <div>
    <div id="render">
      <div class="point" v-for="(point, pointId) in renderData" :key="pointId" :title="pointId"
        v-bind:style="{
          top: point.y * 10 + 'px',
          left: point.x * 10 + 'px',
          transform: 'scale(' + point.value + ')'
        }" />
    </div>
    <canvas id="canvas" width="64" height="48"></canvas>
    <canvas id="backCanvas" width="64" height="48"></canvas>
    <video id="video" width="64" height="48" autoplay></video>
  </div>
</template>

<script>
const base64ImageToRGBMatrix = require('base64-image-utils').base64ImageToRGBMatrix;

function canvasDataToMatrix(data, width, height) {
  const result = []
  for (var y = 0; y < height; y++) {
    result[y] = []
    for (var x = 0; x < width; x++) {
      result[y][x] = {
        r: data[y * width * 4 + x * 4],
        g: data[y * width * 4 + x * 4 + 1],
        b: data[y * width * 4 + x * 4 + 2],
        a: data[y * width * 4 + x * 4 + 3]
      }
    }
  }

  return result
}

function matrixToCanvasData(matrix, width, height) {
  const result = new Array(width * height)

  for (var y = 0; y < height; y++) {
    for (var x = 0; x < width; x++) {
      result[y * width * 4 + x * 4] = matrix[y][x].r;
      result[y * width * 4 + x * 4 + 1] = matrix[y][x].g;
      result[y * width * 4 + x * 4 + 2] = matrix[y][x].b;
      result[y * width * 4 + x * 4 + 3] = matrix[y][x].a;
    }
  }

  return result;
}

function canvasDataToArray(data, width, height) {
  const result = []
  for (var y = 0; y < height; y++) {
    for (var x = 0; x < width; x++) {
      result.push({
        x: x,
        y: y,
        r: data[4 * (y * width + x)],
        g: data[4 * (y * width + x) + 1],
        b: data[4 * (y * width + x) + 2],
        a: data[4 * (y * width + x) + 3]
      })
    }
  }

  return result;
}

function arrayToCanvasData(arrayData, width, height) {
  const result = new Array(width * height * 4)

  for (var y = 0; y < height; y++) {
    for (var x = 0; x < width; x++) {
      result[4 * (y * width+ x)] = arrayData[y * width + x].r,
      result[4 * (y * width+ x) + 1] = arrayData[y * width + x].g,
      result[4 * (y * width+ x) + 2] = arrayData[y * width + x].b,
      result[4 * (y * width+ x) + 3] = arrayData[y * width + x].a
    }
  }

  return result;
}

export default {
  name: 'app',
  data() {
    return {
      canvasCtx: null,
      videoEl: null,
      canvasEl: null,
      renderData: []
    }
  },
  mounted() {
    const renderDataTmp = [];

    for (var x = 0; x < 64; x++) {
      for (var y = 0; y < 48; y++) {
        renderDataTmp[x * 48 + y] = {
          x: x,
          y: y,
          value: 0.25
        }
      }
    }

    this.renderData = renderDataTmp;

    this.videoEl = this.$el.querySelector('#video');
    this.canvasEl = this.$el.querySelector('#canvas');
    this.backCanvasEl = this.$el.querySelector('#backCanvas');

    this.canvasCtx = this.canvasEl.getContext('2d');
    this.backCanvasCtx = this.backCanvasEl.getContext('2d');

    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
        this.videoEl.src = window.URL.createObjectURL(stream);
        this.videoEl.play();
      });
    }

    window.setTimeout(this.snap, 256);
  },
  methods: {
    snap() {
      this.backCanvasCtx.drawImage(this.videoEl, 0, 0, 64, 48);

      var apx = this.backCanvasCtx.getImageData(0, 0, 64, 48);

      const arrayData = canvasDataToArray(apx.data, 64, 48);

      this.renderData = arrayData.map(el => {
        const val = (el.r + el.g + el.b) / (255 * 3);
        return {
          x: el.x,
          y: el.y,
          value: 1 - val
        }
      });

      /*
      RENDER TO CANVAS
      const arrayData = canvasDataToArray(apx.data, 48, 64).map(el => {
        const brillance = el.r + el.g + el.b;

        if (brillance < 90 * 3) {
          el.r = 0;
          el.g = 0;
          el.b = 0;
        } else {
          el.r = 255;
          el.g = 255;
          el.b = 255;
        }

        return el;
      });


      const ctxData = arrayToCanvasData(arrayData, 48, 64);

      for (var w = apx.data.length; w; w--) {
        apx.data[w] = ctxData[w];
      }

      this.canvasCtx.putImageData(apx, 0, 0)
      */

      window.requestAnimationFrame(this.snap)
    }
  }
}
</script>

<style lang="scss">
  * {
    position: relative;
    margin: 0;
    padding: 0
  }

  #video {
  }

  #render {
    width: 640px;
    height:480px;
  }

  #backCanvas, #canvas {
    display: none
  }

  .point {
    position: absolute;
    transform-origin: 50% 50%;
    margin: -7px 0 0 -7px;
    height: 14px;
    width: 14px;
    background: #000;
    border-radius: 10px;
  }
</style>
