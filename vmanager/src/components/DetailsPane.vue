<script setup>
import { resolve, VmState, resolveName, sortIdsByName } from '../model.js'
import StateLabel from './VmState.vue'
defineEmits(['editVm', 'dupVm', 'filterVm', 'rmVm', 'editGroup', 'dupGroup', 'filterGroup', 'rmGroup', 'setState', 'choose'])

const props = defineProps({
  element: Object
})

function list(state) {
  return props.element.members
    .map(v=>resolve(v))
    .filter(vm=>state ? vm.state == state : true)
    .map(o=>o.name).join(' ');
}
</script>

<template>
  <div v-if="element == null || element.id == -1">
    (selecciona una Vm o un grupo para ver sus detalles)
  </div>
  <div v-else-if="Array.isArray(element.groups)">
    <h4>máquina virtual <span class="name">{{ element.name }}</span></h4>

    <table>
      <tr>
        <th>Estado</th>
        <td><StateLabel :state="element.state" /> {{ element.state }} </td>
      </tr>
      <tr>
        <th>Memoria</th>
        <td>{{ element.ram }} Gb</td>
      </tr>
      <tr>
        <th>Disco</th>
        <td>{{ element.hd }} Gb</td>
      </tr>
      <tr>
        <th>Máximo uso de CPU</th>
        <td>{{ element.cpu }} %</td>
      </tr>
      <tr>
        <th>Núcleos de CPU</th>
        <td>{{ element.cores }}</td>
      </tr>
      <tr>
        <th>Dirección IPv4</th>
        <td>{{ element.ip }}</td>
      </tr>
      <tr>
        <th>Ancho de banda máximo</th>
        <td>{{ element.up }} Kbps de subida<br>{{ element.down }} Kbps de bajada</td>
      </tr>
      <tr>
        <th>Disco externo virtual</th>
        <td v-if="element.iso != -1">{{ resolve(element.iso).name }}</td>
        <td v-else> (ninguno) </td>
      </tr>
      <tr>
        <th>Grupos a los que pertenece</th>
        <td v-if="element.groups.length">
          <template v-for="group in sortIdsByName(element.groups).map(g => resolve(g).name)" :key="group">
            <span class="badge text-bg-light" @click="$emit('choose', resolveName(group).id)">
              {{group}}
            </span>
          </template>
        </td>
        <td v-else> (ninguno) </td>
      </tr>
    </table>
  
    <h5>Acciones</h5>
    <div class="btn-group">
      <button @click="$emit('editVm')" :disabled="element.state != VmState.STOPPED" class="btn btn-outline-success" title="Editar máquina virtual">✏️</button>

      <button @click="$emit('dupVm')" class="btn btn-outline-success" title="Duplicar Vm">🖥️</button>

      <button v-if="element.groups.length" class="btn btn-outline-warning" title="Ver grupos que contienen la máquina virtual"
        @click="$emit('filterVm')" >🔬</button>
      
      <button v-if="element.state != VmState.RUNNING" class="btn btn-outline-secondary" title="Iniciar máquina virtual"
        @click="$emit('setState', VmState.RUNNING)" >▶</button>
      <button v-if="element.state != VmState.SUSPENDED" class="btn btn-outline-secondary" title="Suspender máquina virtual"
        @click="$emit('setState', VmState.SUSPENDED)">💤</button>
      <button v-if="element.state != VmState.STOPPED" class="btn btn-outline-secondary" title="Parar máquina virtual"
        @click="$emit('setState', VmState.STOPPED)">🛑</button>
      
      <button @click="$emit('rmVm')" class="btn btn-outline-danger" title="Eliminar máquina virtual">🗑️</button>
    </div>

    </div>

  <div v-else>
    <h4>grupo <span class="name">{{element.name}}</span></h4>

    <b>{{ element.members.length }} integrantes</b>
    <table>
      <tr>
        <th>{{ element.members.length }} integrantes</th>
        <td v-if="element.members.length">
          <template v-for="vm in list(false).split(' ').sort()" :key="vm">
            <span class="badge text-bg-light" @click="$emit('choose', resolveName(vm).id)">
              {{vm}}
            </span>
          </template>
        </td>
        <td v-else> (no hay) </td>
      </tr>
      <tr>
        <th>Encendidas</th>
        <td v-if="list(VmState.RUNNING).length">
          <template v-for="vm in list(VmState.RUNNING).split(' ').sort()" :key="vm">
            <span class="badge text-bg-light" @click="$emit('choose', resolveName(vm).id)">
              {{vm}}
            </span>
          </template>
        </td>
        <td v-else> (no hay) </td>
      </tr>
      <tr>
        <th>Suspendidas</th>
        <td v-if="list(VmState.RUNNING).length">
          <template v-for="vm in list(VmState.SUSPENDED).split(' ').sort()" :key="vm">
            <span class="badge text-bg-light" @click="$emit('choose', resolveName(vm).id)">
              {{vm}}
            </span>
          </template>
        </td>
        <td v-else> (no hay) </td>
      </tr>
      <tr>
        <th>Apagadas</th>
        <td v-if="list(VmState.RUNNING).length">
          <template v-for="vm in list(VmState.STOPPED).split(' ').sort()" :key="vm">
            <span class="badge text-bg-light" @click="$emit('choose', resolveName(vm).id)">
              {{vm}}
            </span>
          </template>
        </td>
        <td v-else> (no hay) </td>
      </tr>
    </table>

    <h5>Acciones</h5>
    <div class="btn-group">
      <button @click="$emit('editGroup')" class="btn btn-outline-success" title="Editar grupo">✏️</button>
      <button @click="$emit('dupGroup')" class="btn btn-outline-success" title="Duplicar Grupo">🖥️</button>
      <button @click="$emit('filterGroup')" class="btn btn-outline-warning" title="Ver máquinas que pertenecen al grupo">🔬</button>
      <!-- Añadido para el Ejercicio 7 -->
      <button v-if="element.members.some(vm => resolve(vm).state != VmState.RUNNING)" class="btn btn-outline-secondary"
        @click="$emit('setState', VmState.RUNNING)" title="Iniciar Grupo">▶</button>
      <button v-if="element.members.some(vm => resolve(vm).state != VmState.SUSPENDED)" class="btn btn-outline-secondary"
        @click="$emit('setState', VmState.SUSPENDED)" title="Suspender Grupo">💤</button>
      <button v-if="element.members.some(vm => resolve(vm).state != VmState.STOPPED)" class="btn btn-outline-secondary"
        @click="$emit('setState', VmState.STOPPED)" title="Parar Grupo">🛑</button>
      <!-- -->
      <button @click="$emit('rmGroup')" class="btn btn-outline-danger" title="Eliminar grupo">🗑️</button>
    </div>
  </div>
</template>

<style scoped>
  span.badge.text-bg-light {
    margin-left: 3px;
    margin-right: 3px;
  }
  tr>th {
    width: 10em;
    text-align: right;
  }
  .name {
    font-weight: 1000;
  }
  td, th {
    padding: 4px;
  }
  h5 {
    margin-top: 1em;
  }
</style>
