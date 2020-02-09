<template>
  <div class="hello">
    <div id="pointers"></div>
    <canvas id="canvas" width="2000" height="1000"></canvas>
  </div>
</template>

<script>
const socket = require("socket.io-client")("http://localhost:3000");

export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  mounted() {
    // socket.on("connect", onConnect);

    function onConnect() {
      console.log("CONNECTED!");
    }

    const url = window.location.origin;
    // console.log(io);
    // let socket = socket(url);
    let prev = {};
    let canvas = document.getElementById("canvas");
    let context = canvas.getContext("2d");
    let pointerContainer = document.getElementById("pointers");

    let pointer = document.createElement("div");
    pointer.setAttribute("class", "pointer");

    let drawing = false;
    let clients = {};
    let pointers = {};

    function drawLine(fromx, fromy, tox, toy) {
      context.moveTo(fromx, fromy);
      context.lineTo(tox, toy);
      context.stroke();
    }

    function now() {
      return new Date().getTime();
    }

    let lastEmit = now();

    canvas.onmouseup = canvas.onmousemove = canvas.onmousedown = function(e) {
      switch (e.type) {
        case "mouseup":
          drawing = false;
          break;

        case "mousemove":
          // if (now() - lastEmit > 50) {
          socket.emit("mousemove", {
            x: e.pageX,
            y: e.pageY,
            drawing: drawing
          });
          lastEmit = now();
          // }

          if (drawing) {
            drawLine(prev.x, prev.y, e.pageX, e.pageY);

            prev.x = e.pageX;
            prev.y = e.pageY;
          }
          break;

        case "mousedown":
          drawing = true;
          prev.x = e.pageX;
          prev.y = e.pageY;
          break;

        default:
          break;
      }
    };

    socket.on("moving", function(data) {
      if (!clients.hasOwnProperty(data.id)) {
        pointers[data.id] = pointerContainer.appendChild(pointer.cloneNode());
      }

      pointers[data.id].style.left = data.x + "px";
      pointers[data.id].style.top = data.y + "px";

      if (data.drawing && clients[data.id]) {
        drawLine(clients[data.id].x, clients[data.id].y, data.x, data.y);
      }

      clients[data.id] = data;
      clients[data.id].updated = now();
    });

    socket.on("clientdisconnect", function(id) {
      delete clients[id];
      if (pointers[id]) {
        pointers[id].parentNode.removeChild(pointers[id]);
      }
    });
  }
};
</script>

<style>
body {
  background: #7aa2b9;
  overflow: hidden;
}

#pointers {
  position: relative;
}

#pointers .pointer {
  position: absolute;
  width: 15px;
  height: 22px;
  background: url("https://uploads.codesandbox.io/uploads/user/88acfe5a-77fc-498c-98ee-d1b0b303f6a8/tC4n-pointer.png")
    no-repeat -4px 0;
}
</style>