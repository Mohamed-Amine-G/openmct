<template>
    <div class="parkingmap">
        <p>Parking Map :: {{ siteName }}</p>
        <div class="map-container">
          <img v-if="mapImage.src" class="map" :src="mapImage.src">
          <svg v-if="mapImage.width" :viewBox="`0 0 ${mapImage.width} ${mapImage.height}`">
            <polygon
              v-for="(slot, index) in slots"
              :key="`slot-${index}`"
              :points="slot['slot location']"
              :style="`fill:${parkings[slot['slot id']] ? 'red' : 'lime'};stroke:purple;stroke-width:1;opacity: 60%;`"
            />
            <g v-for="(camera, index) in cameras" :key="`camera-${index}`" @click="handleClickCamera(camera)">
              <image xlink:href="./camera_button.png" width="62.5" height="83" :x="camera.x" :y="camera.y + 20" />
              <rect :x="camera.x" :y="camera.y" width="62.5" height="20" fill="yellow" />
              <text :x="camera.x" :y="camera.y + 20" margin = "2px" font-size = "12px" font-weight="bold" text-decoration = "underline" >{{camera.name}}</text>
            </g>
          </svg>
        </div>
    </div>
</template>

<script>
import { io } from "socket.io-client";

const socket = io();

export default {
    inject: ['openmct', 'domainObject'],
    data: function () {
      return {
        currentDomainObject: this.domainObject,
        siteName: '',
        slots: [],
        cameras: [],
        parkings: {},
        mapImage: {
          src: '',
          width: 0,
          height: 0
        }
      };
    },
    methods: {
      handleClickCamera(camera) {
        console.log(camera);
      }
    },
    created() {
      try {
        const jsonData = JSON.parse(this.currentDomainObject.jsonFile.body);
        this.siteName = jsonData['site name']; 
        this.slots = jsonData.slots;
        this.cameras = jsonData.cameras.map(cam => ({
          name: cam['camera name'],
          link: cam['camera link'],
          x: parseInt(cam['camera location'][0]),
          y: parseInt(cam['camera location'][1])
        }))
        this.mapImage.src = jsonData['image file name'];
        const img = new Image();
        img.onload = () => {
          const { width, height } = img;
          this.mapImage.width = width;
          this.mapImage.height = height;
        };
        img.src = this.mapImage.src;
      } catch (e) {
        console.log(e);
      }

      socket.on('message', (data) => {
        const cameraData = JSON.parse(data);
        const lots = cameraData['parking lots'];
        for (const slot in lots) {
          this.parkings[slot] = (lots[slot] === 'occupied');
        }
        this.slots = [...this.slots]
      });
    }
};
</script>
