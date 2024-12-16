<template>
  <div id="modal-confirm" v-if="isVisible">
    <div class="modal-content">
      <div class="modal-header">
        <h4>Confirm</h4>
      </div>
      <div class="modal-body">
        <slot name="content"></slot>
      </div>
      <div class="modal-footer">
        <button @click="handleConfirm" class="btClose">Ok</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ModalConfirm",
  props: ["isVisible"],
  methods: {
    handleConfirm() {
      this.$emit("confirm");
    },
  },
};
</script>
<style>
#modal-confirm {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 9999;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
}
.modal-content {
  background: var(--background);
  border: 4px solid var(--grey);
  width: 20rem;
  margin: auto;

  display: grid;
  grid-template-rows: auto 1fr auto;
}
.modal-header {
  padding: 0.5rem;
  display: flex;
  justify-content: space-between;
}
.modal-body {
  padding: 0.5rem;
}

.modal-footer {
  display: flex;
  justify-content: center;
  padding: 0.5rem;
}
.btClose {
  width: 100%;
  padding: 0.25rem 0.75rem;
}
</style>
