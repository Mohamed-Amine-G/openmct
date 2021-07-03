<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal modal--slotinfo" role="dialog">
        <header class="modal-header">
          Parking Slot Information
        </header>
        <section class="modal-body">
          <div class="input-row">
            <label>Slot Name</label>
            <input type="text" v-model="slotName" />
          </div>
          <div class="input-row">
            <label>Slot Id</label>
            <input type="text" v-model="slotId" />
          </div>
          <div class="input-row">
            <label>Slot Owner</label>
            <input type="text" v-model="slotOwner" />
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
    name: 'SlotInfoModal',
    props: ['data'],
    data: function() {
      return {
        slotName: '',
        slotId: '',
        slotOwner: ''
      }
    },
    methods: {
      confirm() {
        this.$emit('confirm', { slotName: this.slotName, slotId: this.slotId, slotOwner: this.slotOwner })
      },
      close() {
        this.slotName = this.data.slotName
        this.slotId = this.data.slotId
        this.slotOwner = this.data.slotOwner
        this.$emit('close');
      }
    }
  };
</script>
