<template>
  <div>
    <div>
      <v-container class="grey lighten-5">
        <v-row>
        Created Players
        </v-row>
        <v-row>
        <Player
          v-for="(player, index) in this.allPlayers"
          :key="index"
          :player="player"
          @removePlayer="removePlayer"
        />
        </v-row>
      </v-container>
    </div>
    <div>
      Create new player
      <div class="container">
        <v-row>
          <v-text-field
            label="Playername"
            v-model="newPlayer.name"
          ></v-text-field>
        </v-row>
        <div class="row">
          <div class="col-md-6">
            <div class="border">
              <web-cam
                ref="webcam"
                :device-id="deviceId"
                width="100%"
                @started="onStarted"
                @stopped="onStopped"
                @error="onError"
                @cameras="onCameras"
                @camera-change="onCameraChange"
              />
            </div>

            <div class="row">
              <div class="col-md-12">
                <select v-model="camera">
                  <option>-- Select Device --</option>
                  <option
                    v-for="device in devices"
                    :key="device.deviceId"
                    :value="device.deviceId"
                    >{{ device.label }}</option
                  >
                </select>
              </div>
            </div>
          </div>
        </div>
      </div>
      <v-btn @click="createNewPlayer">Create new Player</v-btn>
    </div>
  </div>
</template>

<script>
import { WebCam } from "vue-web-cam";
import Player from "@/components/Player";

export default {
  name: "App",
  components: {
    "web-cam": WebCam,
    Player
  },
  data() {
    return {
      img: null,
      camera: null,
      deviceId: null,
      devices: [],
      webcam: null,
      players: [],
      newPlayer: {
        name: "",
        picture: null
      }
    };
  },
  computed: {
    device: function() {
      return this.devices.find(n => n.deviceId === this.deviceId);
    },
    allPlayers(){
      return this.$store.getters.allPlayers;
    },
  },
  watch: {
    camera: function(id) {
      this.deviceId = id;
    },
    devices: function() {
      // Once we have a list select the first one
      const [first, ...tail] = this.devices;
      if (first) {
        this.camera = first.deviceId;
        this.deviceId = first.deviceId;
      }
    }
  },
  mounted() {
    this.webcam = this.$refs.webcam;
  },
  methods: {
    onCapture() {
      this.newPlayer.picture = this.webcam.capture();
    },
    onStarted(stream) {
      console.log("On Started Event", stream);
    },
    onStopped(stream) {
      console.log("On Stopped Event", stream);
    },
    onStop() {
      this.webcam.stop();
    },
    onStart() {
      this.webcam.start();
    },
    onError(error) {
      console.log("On Error Event", error);
    },
    onCameras(cameras) {
      this.devices = cameras;
      console.log("On Cameras Event", cameras);
    },
    onCameraChange(deviceId) {
      this.deviceId = deviceId;
      this.camera = deviceId;
      console.log("On Camera Change Event", deviceId);
    },
    createNewPlayer() {
      this.onCapture();
      this.$store.commit('addPlayer', this.newPlayer);
      /*
      this.onCapture();
      this.players.push(this.newPlayer);
      */
      this.newPlayer = {
        name: "",
        picture: null
      };
    },
    removePlayer(player) {
      this.$store.commit('removePlayer', player)
      /*
      const index = this.players.indexOf(player);
      this.players.splice(index, 1);
      */
    }
  }
};
</script>
