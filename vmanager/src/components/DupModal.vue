<script setup>

import VModal from './VModal.vue'
import RangeBox from './RangeBox.vue'

import * as M from '../model.js'
import { ref } from 'vue'

const emit = defineEmits(['dupVm', 'dupGroup'])

const props = defineProps({
  entity: Object,
  exists: Boolean, // otherwise, do not evaluate isVm
  isVm: Boolean, // otherwise, dup group instead of dup vm
})  

let modalRef = ref(null);

function dupEntity() {
  let entity = props.entity;
  let isVm = props.isVm;
  const form = document.getElementById("dupEntity");
  const valueFor = (name) => {
    const input = form.querySelector(`input[name=${name}]`);
    if (!input) console.log("ERROR: no input for name", name, "in", form);
    return input.value
  }

  entity.id = -1;
  console.log(`saving ${ isVm ? "Vm" : "Group" }...`, entity, form);

  // comprueba validez de todos los campos, y sobreescribe resultado
  if ( ! form.checkValidity()) {
    // fuerza a que se muestren los errores simulando un envío
    // (pero como hay errores, no se va a enviar nada :-)
    form.querySelector("button[type=submit]").click();
    return; 
  }    

  // todo válido: lanza eventos a padre, y cierra modal
  for (let x = 0; x < valueFor("n-copies"); x++) { 
    emit(isVm ? "dupVm" : "dupGroup", entity);
  }
  modalRef.value.hide();
}

// para que el padre pueda llamar a show (hide no debería hacer falta)
function show() {
  modalRef.value.show();
}
defineExpose({ show });
</script>

<template>
  <VModal ref="modalRef" id="DupModal"
    :title= "`Duplicando: ${isVm ? 'Vm' : 'Grupo'} ${entity.name}`" >
    <template #body>
      <form id="dupEntity" 
        @submit.prevent="e => dupEntity()">
        <div class="container">
          <RangeBox start="1" id="n-copies" label="Copies" :min="1" :max="1000" :units="isVm ? 'Vms' : 'Grupos'" />
        </div>
        <button type="submit" class="invisible">Submit</button>
      </form>
    </template>
    <template #footer>
      <button @click.prevent="() => dupEntity()" class="btn btn-primary">
        {{ `Duplicar ${entity.name}` }}
      </button>
    </template>
  </VModal>
</template>

<style scoped>
</style>