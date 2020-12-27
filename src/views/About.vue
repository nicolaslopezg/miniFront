<template>
  <div class="about">
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
          <h3 class="panel-title">Active Users:</h3>
        </div>
        <div class="video-chat-container">
          <h2 class="talk-info" id="talking-with-info">
            Select active user on the left menu.
          </h2>
          <div class="video-container">
            <video v-if="renderRemoteVideo" autoplay class="remote-video" id="remote-video"></video>
            <video autoplay muted class="local-video" id="local-video"></video>
          </div>
        </div>
      </div>
    </div>
    <button @click="endOperator()">Cortar</button>
  </div>
</template>

<script>

import SocketIO from 'socket.io-client';

const { RTCPeerConnection, RTCSessionDescription } = window;

export default {
  name: 'About',
  props: {
    msg: String,
  },
  data() {
    return {
      // clientSocket: SocketIO('http://67.205.168.127:3001/clients/clientsQueue', {
      clientSocket: SocketIO('http://localhost:3000/clients/clientsQueue', {
        transports: ['websocket'],
        useConnectionNamespace: true,
      }),
      peerConnection: new RTCPeerConnection({
        configuration: {
          offerToReceiveAudio: true,
          offerToReceiveVideo: true,
        },
        iceServers: [],
      }),
      renderRemoteVideo: false,
    };
  },
  methods: {
    async joinVideoCallRoom(data) {
      console.log('Operador asignado!');
      console.log(data);
      // TODO: Redirect to room with video
      await this.peerConnection.setRemoteDescription(
        new RTCSessionDescription(data.offer),
      );
      const answer = await this.peerConnection.createAnswer();
      await this.peerConnection.setLocalDescription(new RTCSessionDescription(answer));
      this.clientSocket.emit('client-answer-operator', {
        answer,
        operatorId: data.operatorId,
      });
    },
    async answerOperator(data) {
      try {
        console.log('[Answer operator]');
        console.log(data);
      } catch (e) {
        console.log('Error => ', e);
      }
    },
    async endOperator() {
      this.$router.push('/greetings');
      this.clientSocket.emit('redirect-operator');
      this.clientSocket.emit('disconnect-client');
    },
    async redirectClient() {
      this.$router.push('/greetings');
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
            this.renderRemoteVideo = true;
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
    this.clientSocket.emit('client-request', {
      fullName: 'Sebastián Salazar',
      dni: '22.805.902-1',
      email: 'seba.salazar.vivanco@gmail.com',
      phoneNumber: '+56 962261873',
      status: 'waiting',
    });
    this.clientSocket.on('operator-attend-client', this.joinVideoCallRoom);
    this.clientSocket.on('operator-answer-client', this.answerOperator);
    this.clientSocket.on('disconnect');
    this.clientSocket.on('redirect-client', this.redirectClient);
    this.getLocalVideo();
    this.getRemoteVideo();
  },
};

</script>
