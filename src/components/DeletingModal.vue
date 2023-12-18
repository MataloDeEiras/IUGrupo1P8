<script setup>

import VModal from './VModal.vue'

import * as M from '../model.js'
import { ref } from 'vue'

const emit = defineEmits(['dlVm', 'dlGroup'])

const props = defineProps({
  vmOrg: Object,
  isVM: Boolean, // otherwise, deleting group
})

let modalRef = ref(null);

function rmObj(confirmed) {
  const obj = props.vmOrg;

  console.log("removing...", obj)

  if (confirmed) emit(props.isVM ? 'dlVm' : 'dlGroup', obj.id)

  console.log("passed...", obj)

  modalRef.value.hide()    
}

// para que el padre pueda llamar a show (hide no debería hacer falta)
function show() {
  modalRef.value.show();
}
defineExpose({ show });
</script>

<template>
  <VModal ref="modalRef" id="deletingModal"
  :title="isVM ? `Eliminar VM: ${vmOrg.name}` : `Eliminar Grupo: ${vmOrg.name}`" >
    <template #body>
      <form id="deleting"
      @submit.prevent="e => rmObj(false)">
        <div class="container">
          Esta acción no se podrá deshacer, ¿estás seguro de que quieres eliminar {{vmOrg.name}}?
        </div>
        <button type="submit" class="invisible">Submit</button>
      </form>
    </template>
    <template #footer>
      <button @click.prevent="() => rmObj(true)" class="btn btn-danger">
        {{ `Eliminar ${vmOrg.name}` }}
      </button>
    </template>
  </VModal>
</template>

<style scoped>
</style>