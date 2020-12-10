<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    {{ clients }}
    <button @click="attendClient(clients[0])">Atender</button>
  </div>
</template>

<script>

import SocketIO from 'socket.io-client';

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  data() {
    return {
      // socket: SocketIO('http://67.205.168.127:555/operators/clientsQueue', {
      socket: SocketIO('http://localhost:3001/operators/clientsQueue', {
        transports: ['websocket'],
        useConnectionNamespace: true,
      }),
      clients: [],
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
    },
    setClients(clients) {
      console.log('Setear clientes!');
      console.log(JSON.parse(clients));
      this.clients = JSON.parse(clients);
    },
    attendClient(client) {
      if (!client) {
        return;
      }
      this.socket.emit('attend-client', client);
    },
  },
  mounted() {
    this.socket.on('new-client', this.newClient);
    this.socket.on('remove-client', this.removeClient);
    this.socket.on('first-clients-fetch', this.setClients);
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
