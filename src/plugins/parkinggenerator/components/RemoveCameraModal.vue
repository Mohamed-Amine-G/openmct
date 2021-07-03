<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal modal--removecamera" role="dialog">
        <header class="modal-header">
          Remove Camera
        </header>
        <section class="modal-body">
          <h6>Are you sure remove camera box: {{ data.cameraName }}?</h6>
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
    name: 'RemoveCameraModal',
    props: ['data'],
    methods: {
      confirm() {
        this.$emit('confirm', { cameraName: this.data.cameraName })
      },
      close() {
        this.$emit('close');
      }
    }
  };
</script>
