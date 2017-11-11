<template>
  <div id="app">
    <input type="text" v-model.trim="messageInput" @keyup.enter="send(messageInput)">
    <button @click="init()" type="button" name="button">Cek</button>
    <div v-for="msg in messages">
      <p>{{msg.message}} lat: {{msg.lat}} lon: {{msg.lon}}</p>
    </div>

    <gmap-map
    :center="center"
    :zoom="7"
    style="width: 500px; height: 300px">

    <gmap-marker
      :key="index"
      v-for="(m, index) in markers"
      :position="m.position"
      :clickable="true"
      :draggable="true"
      @click="center=m.position">
    </gmap-marker>
    </gmap-map>

    <router-view/>
  </div>
</template>

<script>
import * as Firebase from 'firebase'
import Geohash from 'latlon-geohash'
import geo from 'geo-hash'
export default {
  name: 'app',
  data () {
    return {
      room: null,
      precision: 6,
      db: null,
      messageInput: '',
      messages: [],
      center: {lat: 10.0, lng: 10.0},
      // markers: [{
      //   position: {lat: 10.0, lng: 10.0}
      // }, {
      //   position: {lat: 11.0, lng: 11.0}
      // }],
      markers: [],
      tampungan: []
    }
  },
  methods: {
    send (messageInput) {
      // this.init()
      navigator.geolocation.getCurrentPosition((position) => {
        var geohash = Geohash.encode(position.coords.latitude, position.coords.longitude, this.precision)
        console.log(geohash)
        this.room = geo.decode(geohash)
        let data = {
          message: messageInput,
          lat: this.room.lat,
          lon: this.room.lon
        }
        this.db.database().ref('messages/').push(data)
        this.messageInput = ''
      }, (err) => {
        console.log(err)
      })
    },
    messageListener () {
      console.log('work')
      this.db.database().ref('messages').on('value', (snap) => {
        this.messages = snap.val()
        // console.log(this.messages)
        for (let key in this.messages) {
          let obj = {
            position: {
              lat: this.messages[key].lat,
              lng: this.messages[key].lon
            }
          }
          this.markers.push(obj)
        }
        console.log(this.tampungan)
      })
    }
  },
  mounted () {
    this.db = Firebase.initializeApp({
      apiKey: 'AIzaSyCfUDFkgzFt_Nzq7hJjX_DQmQE5KuNXw3g',
      authDomain: 'websocket-slide-ba8fd.firebaseapp.com',
      databaseURL: 'https://websocket-slide-ba8fd.firebaseio.com',
      projectId: 'websocket-slide-ba8fd',
      storageBucket: 'websocket-slide-ba8fd.appspot.com',
      messagingSenderId: '258066709342'
    })
    this.messageListener()
  },
  watch: {
    markers: function (data) {
      console.log('baru')
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /*margin-top: 60px;*/
}
</style>
