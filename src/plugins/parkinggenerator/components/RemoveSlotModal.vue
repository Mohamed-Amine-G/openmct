<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal modal--removeslot" role="dialog">
        <header class="modal-header">
          Remove Slot Box
        </header>
        <section class="modal-body">
          <h6>Are you sure remove slot box: {{ data.slotName }}?</h6>
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
    name: 'RemoveSlotModal',
    props: ['data'],
    methods: {
      confirm() {
        this.$emit('confirm', { slotId: this.data.slotId })
      },
      close() {
        this.$emit('close');
      }
    }
  };
</script>
