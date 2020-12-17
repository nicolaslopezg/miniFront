<template>
  <div class="hello">
    <div class="container">
      <header class="header">
        <div class="logo-container">
          <h1 class="logo-text">
            Doge<span class="logo-highlight">ller</span>
          </h1>
        </div>
      </header>
      <div class="content-container">
        <div class="active-users-panel" id="active-user-container">
          <h3 class="panel-title">Active Users: {{ clients }}</h3>
        </div>
        <div class="video-chat-container">
          <h2 class="talk-info" id="talking-with-info">
            Select active user on the left menu.
          </h2>
          <div class="video-container">
            <video autoplay class="remote-video" id="remote-video"></video>
            <video autoplay muted class="local-video" id="local-video"></video>
          </div>
        </div>
      </div>
    </div>
    <button @click="attendClient(clients[0])">Atender</button>
    <button @click="endClient(clients[0])">Cortar</button>
  </div>
</template>

<script>

import SocketIO from 'socket.io-client';

const { RTCPeerConnection, RTCSessionDescription } = window;

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  data() {
    return {
      // socket: SocketIO('http://67.205.168.127:555/operators/clientsQueue', {
      socket: SocketIO('http://localhost:3000/operators/clientsQueue', {
        transports: ['websocket'],
        useConnectionNamespace: true,
      }),
      clients: [],
      peerConnection: new RTCPeerConnection({
        configuration: {
          offerToReceiveAudio: true,
          offerToReceiveVideo: true,
        },
        iceServers: [],
      }),
    };
  },
  methods: {
    newClient(newClient) {
      console.log('Nuevo cliente!');
      console.log(newClient);
      this.clients.push(newClient);
    },
    removeClient(clientId) {
      console.log('Se nos fue un cliente :/');
      console.log(clientId);
      this.clients = this.clients.filter((client) => client.id !== clientId);
    },
    setClients(clients) {
      console.log('Setear clientes!');
      console.log(JSON.parse(clients));
      this.clients = JSON.parse(clients);
    },
    async attendClient(client) {
      if (!client) {
        return;
      }
      console.log(client);
      const offer = await this.peerConnection.createOffer();
      await this.peerConnection.setLocalDescription(new RTCSessionDescription(offer));
      this.socket.emit('operator-attend-client', {
        offer,
        clientId: client.id,
      });
    },
    async endClient(client) {
      if (!client) {
        return;
      }
      console.log(client);
      this.$router.push('/vista-panel');
      this.socket.emit('redirect-client');
      this.socket.emit('operator-endCall', client);
    },
    async answerClient(data) {
      console.log(data);
      await this.getLocalVideo();
      await this.peerConnection.setRemoteDescription(
        new RTCSessionDescription(data.answer),
      );
    },
    async redirectOperator() {
      this.$router.push('/vista-panel');
    },
    getLocalVideo() {
      try {
        if (navigator?.mediaDevices) {
          navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
            const localVideo = document.getElementById('local-video');
            if (localVideo) {
              console.log(stream);
              localVideo.srcObject = stream;
            }
            stream.getTracks().forEach((track) => {
              console.log(track);
              this.peerConnection.addTrack(track, stream);
            });
          });
        }
      } catch (e) {
        console.log('Error => ', e);
      }
    },
    getRemoteVideo() {
      try {
        this.peerConnection.ontrack = ({ streams: [stream] }) => {
          console.log('peerConnection new track');
          const remoteVideo = document.getElementById('remote-video');
          if (remoteVideo) {
            console.log(stream);
            remoteVideo.srcObject = stream;
          }
        };
      } catch (e) {
        console.log('Error => ', e);
      }
    },
  },
  mounted() {
    this.socket.on('new-client', this.newClient);
    this.socket.on('remove-client', this.removeClient);
    this.socket.on('first-clients-fetch', this.setClients);
    this.socket.on('client-answer-operator', this.answerClient);
    this.socket.on('redirect', this.redirectOperator);
    this.getLocalVideo();
    this.getRemoteVideo();
  },
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
