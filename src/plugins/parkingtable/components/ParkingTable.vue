<template>
    <div class="parkingtable">
        <p>Parking Table :: {{ siteName }}</p>
        <div class="list">
          <table>
              <thead>
                <tr>
                  <th class="no">No.</th>
                  <th class="slot_name">Slot Name</th>
                  <th class="slot_owner">Slot Owner</th>
                  <th class="parking_start_time">Parking Start Time</th>
                  <th class="parking_time">Parking Time</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in logs" :key="`log-${index}`" :style="[parkings[item['slot name']].empty?{'background-color':'#0096ff'}:{'background-color':'#f27200'}]">
                  <td>{{index + 1}}</td>
                  <td>{{item['slot name']}}</td>
                  <td>{{item['slot owner']}}</td>
                  <td>{{parkings[item['slot name']].empty ? 'Empty' : 'Occupied'}}</td>
                  <td>{{parkings[item['slot name']].duration || '00:00'}}</td>
                </tr>
            </tbody>
          </table>
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
            logs: [],
            parkings: {}
        };
    },
    created() {
      try {
        const jsonData = JSON.parse(this.currentDomainObject.jsonFile.body);
        this.siteName = jsonData['site name'];
        jsonData.slots.forEach((slot) => {
          this.parkings[slot['slot name']] = { empty: true }
        });
        this.logs = jsonData.slots;
      } catch (e) {
        console.log(e);
      }

      socket.on('message', (data) => {
        try {
          const cameraData = JSON.parse(data);
          const lots = cameraData['parking lots'];
          for (const slot in lots) {
            if (!this.parkings[slot]) continue;
            if (lots[slot] === 'occupied') {
              if (this.parkings[slot].empty) {
                this.parkings[slot].empty = false;
                this.parkings[slot].timestamp = new Date();
              }
            } else {
              this.parkings[slot] = { empty: true };
            }
          }
          this.logs = [...this.logs]
        } catch (e) {
          console.log(e);
        }
      });
      this.durationTimer = setInterval(() => {
        for (const slot in this.parkings) {
          if (this.parkings[slot].timestamp) {
            let diff = new Date() - this.parkings[slot].timestamp;
            diff = Math.floor(diff / 1000);
            const min = Math.floor(diff / 60) % 60;
            const sec = diff % 60;
            this.parkings[slot].duration = `${min >= 10 ? '' : 0}${min}:${sec >= 10 ? '' : 0}${sec}`;
          }
        }
        this.logs = [...this.logs]
      }, 500);
    },
    destroyed() {
      clearInterval(this.durationTimer);
    }
};
</script>