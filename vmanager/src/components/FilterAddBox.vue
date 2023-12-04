<script setup>
import { ref } from 'vue'

const props = defineProps([
  'modelValue', 
  'addBtnTitle',
  //Añadido por Galdo para el Ejercicio 7
  'runBtnTitle',
  'slpBtnTitle',
  'stpBtnTitle',
  //
  'cols'
])

//Añadido desde runGroup por Galdo para el Ejercicio 7
const emit = defineEmits(['update:modelValue', 'addElement', 'runGroup', 'sleepGroup', 'stopGroup'])

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
    <!-- Añadido por Galdo para el Ejercicio 7 -->
    <div class="col-auto d-inline">
      <button type="button" :title="runBtnTitle"
      @click="$emit('runGroup')"
      class="btn btn-outline-success">▶</button>
      <button type="button" :title="slpBtnTitle"
      @click="$emit('sleepGroup')"
      class="btn btn-outline-warning">🌙</button>
      <button type="button" :title="stpBtnTitle"
      @click="$emit('stopGroup')"
      class="btn btn-outline-danger">⬛</button>
    </div>
    <!-- -->
  </div>
  <div v-if="advSearch" class="row mt-3">
    <div class="col-auto">
      Filtros por campos:
      {{ cols }}
    </div>
  </div>
</template>

<style scoped>
.btn.active.b-avanzada {
  background-color: lightblue;
}
</style>
