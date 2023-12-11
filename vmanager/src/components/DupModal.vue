<script setup>

import VModal from './VModal.vue'
import RangeBox from './RangeBox.vue'

import * as M from '../model.js'
import { ref } from 'vue'

const emit = defineEmits(['dup'])

const props = defineProps({
  vm: Object,
  isAdd: Boolean, // otherwise, editing existing instead of adding
})  

let modalRef = ref(null);

function dupVm() {
  let vm = props.vm;
  const form = document.getElementById("dupVm");
  const valueFor = (name) => {
    const input = form.querySelector(`input[name=${name}]`);
    if (!input) console.log("ERROR: no input for name", name, "in", form);
    return input.value
  }

  vm.id = -1;
  console.log("saving VM...", vm, form);

  // comprueba validez de todos los campos, y sobreescribe resultado
  if ( ! form.checkValidity()) {
    // fuerza a que se muestren los errores simulando un envío
    // (pero como hay errores, no se va a enviar nada :-)
    form.querySelector("button[type=submit]").click();
    return; 
  }    

  // todo válido: lanza eventos a padre, y cierra modal
  for (let x = 0; x < valueFor("n-copies"); x++) { 
    emit('dup', vm);
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
    :title= "`Duplicando: ${vm.name}`" >
    <template #body>
      <form id="dupVm" 
        @submit.prevent="e => dupVm()">
        <div class="container">
          <RangeBox start="1" id="n-copies" label="Copies" :min="1" :max="1000" units="Vms" />
        </div>
        <button type="submit" class="invisible">Submit</button>
      </form>
    </template>
    <template #footer>
      <button @click.prevent="() => dupVm()" class="btn btn-primary">
        {{ `Duplicar ${vm.name}` }}
      </button>
    </template>
  </VModal>
</template>

<style scoped>
</style>