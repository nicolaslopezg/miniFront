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
            <video autoplay class="remote-video" id="remote-video"></video>
            <video autoplay muted class="local-video" id="local-video"></video>
          </div>
        </div>
      </div>
    </div>
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
      clientSocket: SocketIO('http://localhost:3001/clients/clientsQueue', {
        transports: ['websocket'],
        useConnectionNamespace: true,
      }),
      peerConnection: new RTCPeerConnection(),
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
      this.getRemoteVideo();
    },
    async answerOperator(data) {
      try {
        console.log(data);
        await this.peerConnection.setRemoteDescription(
          new RTCSessionDescription(data.answer),
        );
      } catch (e) {
        console.log('Error => ', e);
      }
    },
    async getLocalVideo() {
      try {
        console.log(navigator.mediaDevices.getUserMedia);
        let stream = null;
        if (navigator?.mediaDevices) {
          stream = await navigator.mediaDevices.getUserMedia({ video: true });
          const localVideo = document.getElementById('local-video');
          if (localVideo) {
            localVideo.srcObject = stream;
          }
          stream.getTracks().forEach((track) => {
            console.log(track);
            this.peerConnection.addTrack(track, stream);
          });
        }
      } catch (e) {
        console.log('Error => ', e);
      }
    },
    async getRemoteVideo() {
      try {
        this.peerConnection.ontrack = ({ streams: [stream] }) => {
          console.log('peerConnection new track');
          const remoteVideo = document.getElementById('remote-video');
          console.log(stream);
          if (remoteVideo && window.URL) {
            remoteVideo.srcObject = window.URL.createObjectURL(stream);
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
    });
    this.clientSocket.on('operator-attend-client', this.joinVideoCallRoom);
    this.clientSocket.on('operator-answer-client', this.answerOperator);
    this.getLocalVideo();
  },
};

</script>
