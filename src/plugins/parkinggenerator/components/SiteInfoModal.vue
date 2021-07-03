<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal modal--siteinfo" role="dialog">
        <header class="modal-header">
          Set Site Information
        </header>
        <section class="modal-body">
          <div class="input-row">
            <label>Site Name</label>
            <input type="text" v-model="siteName" />
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
    name: 'SiteInfoModal',
    props: ['data'],
    data: function() {
      return {
        siteName: this.data.siteName
      }
    },
    methods: {
      confirm() {
        this.$emit('confirm', { siteName: this.siteName })
      },
      close() {
        this.siteName = this.data.siteName
        this.$emit('close');
      }
    }
  };
</script>
