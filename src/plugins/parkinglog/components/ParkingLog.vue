<template>
    <div class="parkinglog">
        <form action="">
          <div class="searchbox">
              <label for="startTime">From date/time</label><br>
              <input type="date" id="startDate" name="startDate" placeholder="YYYY-MM-DD" v-model="form.startDate">
              <input type="time" id="startTime" name="startTime" placeholder="HH:MM:SS" v-model="form.startTime">
          </div>
          <div class="searchbox">
              <label for="endTime">To date/time</label><br>
              <input type="date" id="endDate" name="endDate" placeholder="YYYY-MM-DD" v-model="form.endDate">
              <input type="time" id="endTime" name="endTime" placeholder="HH:MM:SS" v-model="form.endTime">
          </div>
          <div class="searchbox">
              <label for="slotName">Slot name/No</label><br>
              <input type="text" id="slotName" name="slotName" v-model="form.slotName">
          </div>
          <button type="button" @click="handleShowLog">SHOW LOG</button>
          <button type="button" @click="handleClearLog">CLEAR LOG</button>
        </form>
        <div class="list">
          <table>
            <thead>
              <tr>
                <th class="Event">Event No</th>
                <th class="time">Time</th>
                <th class="slot">Slot Name/Number</th>
                <th class="state">State of slot</th>
                <th>Photo</th>
              </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in logs" :key="`log-${index}`">
                  <td>{{index + 1}}</td>
                  <td>{{item['Time']}}</td>
                  <td>{{item['SLOT NAME']}}</td>
                  <td>{{item['STATUS']}}</td>
                  <td><img :src="`http://localhost:5000/static${item['PHOTOS']}`"></td>
                </tr>
            </tbody>
          </table>
        </div>
    </div>
</template>

<script>
export default {
    inject: ['openmct', 'domainObject'],
    data: function () {
        return {
            currentDomainObject: this.domainObject,
            form: {
                startDate: '',
                startTime: '',
                endDate: '',
                endTime: '',
                slotName: ''
            },
            logs: []
        };
    },
    methods: {
      handleShowLog() {
          const start = !!this.form.startDate && !!this.form.startTime ? `${this.form.startDate} ${this.form.startTime}` : '';
          const end = !!this.form.endDate && !!this.form.endTime ? `${this.form.endDate} ${this.form.endTime}` : '';
          fetch(`http://localhost:5000/data?START_DATE=${start}&END_DATE=${end}&SLOT_NAME=${this.form.slotName}`, {method: 'GET', redirect: 'follow'})
            .then(response => response.json())
            .then(result => this.logs = result)
            .catch(error => console.log('error', error));
      },
      handleClearLog() {
          this.logs = [];
      }
    }
};
</script>
