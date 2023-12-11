<script setup>
import { ref } from 'vue'
//Añadido para el Ejercicio 7
import { VmState } from '../model.js';

const props = defineProps([
  'modelValue', 
  'addBtnTitle',
  //Añadido para el Ejercicio 7
  'runBtnTitle',
  'suspendBtnTitle',
  'stopBtnTitle',
  //
  'cols'
])

//Añadido setState para el Ejercicio 7
const emit = defineEmits(['update:modelValue', 'addElement', 'setState'])

const searchKey = ref('')
const advSearch = ref(false)
const advSearchQuery = ref(['foo'])

function send() {
  emit('update:modelValue', {
    all: searchKey.value, 
    fields: advSearch.value ? advSearchQuery.value : []
  })
}

</script>

<template>
  <div class="row">
    <div class="col-auto w-75">
      <div class="input-group">
        <input 
          :value="searchKey"
          @input="{ searchKey = $event.target.value; send()}" 
          type="search" class="form-control" placeholder="Filtrar">
        <span class="input-group-text btn-outline-secondary">🔍</span>
        <button type="button" 
          data-bs-toggle="button" 
          class="input-group-text btn btn-outline-secondary b-avanzada"
          @click="{ advSearch = !advSearch; send()}" 
          title="Búsqueda avanzada">⚙️</button>
      </div>
    </div>
    <div class="col-auto">

    </div>
    <div class="col-auto">
      <button type="button" :title="addBtnTitle" 
      @click="$emit('addElement')"
      class="btn btn-outline-primary">➕</button>
    </div>
    <!-- Añadido para el Ejercicio 7 -->
    <div class="col-auto d-inline btn-group">
      <button type="button" :title="runBtnTitle"
      @click="$emit('setState', VmState.RUNNING)"
      class="btn btn-outline-success">▶</button>
      <button type="button" :title="suspendBtnTitle"
      @click="$emit('setState', VmState.SUSPENDED)"
      class="btn btn-outline-warning">💤</button>
      <button type="button" :title="stopBtnTitle"
      @click="$emit('setState', VmState.STOPPED)"
      class="btn btn-outline-danger">🛑</button>
    </div>
    <!-- -->
  </div>
  <div v-if="advSearch" class="row mt-3">
    <div class="col-auto">
      Filtros por campos:
      <span class="bold">{{ cols.join(", ") }}</span>
    </div>
  </div>
</template>

<style scoped>
  .bold {
    font-weight: 700;
  }
  .btn.active.b-avanzada {
    background-color: lightblue;
  }
</style>