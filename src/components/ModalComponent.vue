<script setup lang="ts">
import { Modal } from 'bootstrap';
import {onMounted, watch} from "vue";

const props = defineProps<{ text: string, show: boolean }>();

const emit = defineEmits(['hidden']);

let bootstrapModal: Modal;

onMounted(() => {
  const modalElement = document.getElementById('myModal') as HTMLElement;
  bootstrapModal = new Modal(modalElement);

  if (props.show) {
    bootstrapModal.show();
  }

  modalElement.addEventListener('hidden.bs.modal', () => {
    emit('hidden');
  });
});

watch(() => props.show, (show) => {
  if (show) {
    bootstrapModal.show();
  }
});
</script>

<template>
  <div class="modal fade" id="myModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h1 class="modal-title fs-5" id="exampleModalLabel">Modal title</h1>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          {{ text }}
        </div>
        <div class="modal-footer">
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>
