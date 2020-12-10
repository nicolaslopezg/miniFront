<template>
  <div class="about">
    <h1>This is an about page</h1>
  </div>
</template>

<script>

import SocketIO from 'socket.io-client';

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
    };
  },
  mounted() {
    this.clientSocket.emit('client-request', {
      fullName: 'Sebastián Salazar',
      dni: '22.805.902-1',
      email: 'seba.salazar.vivanco@gmail.com',
      phoneNumber: '+56 962261873',
    });
  },
};

</script>
