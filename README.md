const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static(__dirname));

io.on("connection", (socket) => {
  socket.on("offer", data => socket.broadcast.emit("offer", data));
  socket.on("answer", data => socket.broadcast.emit("answer", data));
  socket.on("ice-candidate", data => socket.broadcast.emit("ice-candidate", data));
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => console.log("Server running"));
{
  "name": "remote-phone",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "socket.io": "^4.7.2"
  }
}
<!DOCTYPE html>
<html>
<body>
  <h2>📱 الشاشة</h2>
  <video id="remoteVideo" autoplay playsinline style="width:100%"></video>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();
    const pc = new RTCPeerConnection();

    pc.ontrack = (event) => {
      document.getElementById("remoteVideo").srcObject = event.streams[0];
    };

    socket.on("offer", async (offer) => {
      await pc.setRemoteDescription(offer);
      const answer = await pc.createAnswer();
      await pc.setLocalDescription(answer);
      socket.emit("answer", answer);
    });

    socket.on("ice-candidate", (candidate) => {
      pc.addIceCandidate(candidate);
    });

    pc.onicecandidate = (event) => {
      if (event.candidate) {
        socket.emit("ice-candidate", event.candidate);
      }
    };
  </script>
</body>
</html>
<!DOCTYPE html>
<html>
<body>
  <h2>📤 مشاركة الشاشة</h2>
  <button onclick="start()">Start</button>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();
    const pc = new RTCPeerConnection();

    async function start() {
      const stream = await navigator.mediaDevices.getDisplayMedia({ video: true });
      stream.getTracks().forEach(track => pc.addTrack(track, stream));

      pc.onicecandidate = (event) => {
        if (event.candidate) {
          socket.emit("ice-candidate", event.candidate);
        }
      };

      const offer = await pc.createOffer();
      await pc.setLocalDescription(offer);
      socket.emit("offer", offer);
    }

    socket.on("answer", answer => {
      pc.setRemoteDescription(answer);
    });
  </script>
</body>
</html>
