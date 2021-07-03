<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal modal--camerainfo" role="dialog">
        <header class="modal-header">
          Camera Information
        </header>
        <section class="modal-body">
          <div class="input-row">
            <label>Camera Name</label>
            <input type="text" v-model="cameraName" />
          </div>
          <div class="input-row">
            <label>Camera Link URL</label>
            <input type="text" v-model="cameraLink" />
          </div>
        </section>
        <footer class="modal-footer">
          <button type="button" @click="confirm">Confirm</button>
          <button type="button" @click="close">Close</button>
        </footer>
      </div>
    </div>
  </transition>
</template>

<script>
  export default {
    name: 'CameraInfoModal',
    props: ['data'],
    data: function() {
      return {
        cameraName: '',
        cameraLink: ''
      }
    },
    methods: {
      confirm() {
        this.$emit('confirm', { cameraName: this.cameraName, cameraLink: this.cameraLink })
      },
      close() {
        this.cameraName = this.data.cameraName
        this.cameraLink = this.data.cameraLink
        this.$emit('close');
      }
    }
  };
</script>
