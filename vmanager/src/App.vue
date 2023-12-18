<script setup>
import * as M from './model.js'

import VmGrid from './components/VmGrid.vue'
import FilterAddBox from './components/FilterAddBox.vue'
import VmAddOrEditModal from './components/VmAddOrEditModal.vue'
import GroupAddOrEditModal from './components/GroupAddOrEditModal.vue'
import DupModal from './components/DupModal.vue'
import DeletingModal from './components/DeletingModal.vue'
import DetailsPane from './components/DetailsPane.vue'

import { ref, onMounted, nextTick, isVNode } from 'vue'
import * as bootstrap from 'bootstrap'

// inicializa modelo (si esto fuese de verdad, habría un login previo)
M.init();
// trampas: da acceso al modelo desde la consola
window.M = M

// las partes que nos interesan
const vms = ref(M.getVms());
const groups = ref(M.getGroups());
const selected = ref({id: -1});
let checkboxed_vms = [];
let checkboxed_groups = [];

// tooltips
onMounted(() => {
  // see https://getbootstrap.com/docs/5.3/components/tooltips/#overview
  [...document.querySelectorAll('[data-bs-toggle="tooltip"]')]
    .map(tooltipTriggerEl => new bootstrap.Tooltip(tooltipTriggerEl))
})

// recarga vistas
function refresh() {
  vms.value = M.getVms();
  groups.value = M.getGroups();
  if (selected.value.id != -1) {
    selected.value = M.resolve(selected.value.id);
  }
}

/////
// Vms
/////

// modal para añadir/editar vms
let vmModalRef = ref(null);
// modal para duplicar entidades
let dupModalRef = ref(null);
// modal para borrar entidades
let delModalRef = ref(null);
const defaultNewVm = new M.Vm(-1, 'nueva máquina', 4, 100, 50, 1,
        "0.0.0.0", 100, 100, -1, M.VmState.STOPPED, 100, 100, []);
let vmToAddOrEdit = ref(defaultNewVm);
let entityDup = ref(defaultNewVm);
const deleteVmOrGroup = [-1, false]
let vmOrGroupToDelete = ref(deleteVmOrGroup);



// empieza a editar una Vm; pasa -1 para crear una nueva
async function edVm(id) {
  console.log("now editing", id)
  vmToAddOrEdit.value = (id == -1) ?
    vmToAddOrEdit.value = defaultNewVm :
    vmToAddOrEdit.value = M.resolve(id);
  // da tiempo a Vue para que prepare el componente antes de mostrarlo
  await nextTick()
  vmModalRef.value.show()
}

async function dupVm(id) { //Se duplica con el MISMO nombre
  entityDup.value = M.resolve(id);
  console.log('duping vm', id);
  await nextTick();
  dupModalRef.value.show();
}

async function rmVm(id) {
  console.log("now deleting", id)
  vmOrGroupToDelete.value = (id == -1) ? [null, false] : [ M.resolve(id), true];

  // da tiempo a Vue para que prepare el componente antes de mostrarlo
  await nextTick()
  delModalRef.value.show()
 // M.rmVm(id);
 // if (selected.value.id == id) {
 //   selected.value = {id: -1};
 // }
 // refresh();
}

function updateVmGroups(id) {
  let groups = M.resolve(id).groups;
  for (let index = 0; index < groups.length; index++) {
    let gr = M.resolve(groups[index]);
    let hasMember = false;
    for (let vm_index = 0; vm_index < gr.members.length; vm_index++) {
      if (gr.members[vm_index] == id) {
        hasMember = true;
      }
    }

    if (!hasMember) {
      gr.members.push(id);
    }
  }
}

function removeVmGroups(id) {
  let groups = M.resolve(id).groups;
  for (let index = 0; index < groups.length; index++) {
    let gr = M.resolve(groups[index]);
    for (let vm_index = 0; vm_index < gr.members.length; vm_index++) {
      if (gr.members[vm_index] == id) {
        gr.members.splice(vm_index, 1);
      }
    }
  }
}

/////
// Grupos
/////

// modal para añadir/editar grupos
let groupModalRef = ref(null);
const defaultNewGroup = new M.Group(-1, 'nuevo grupo', []);
let groupToAddOrEdit = ref(defaultNewGroup);

// empieza a editar un grupo; pasa -1 para crear uno nuevo
async function edGroup(id) {
  console.log("now editing", id)
  groupToAddOrEdit.value = (id == -1) ?
    groupToAddOrEdit.value = defaultNewGroup :
    groupToAddOrEdit.value = M.resolve(id);
  // da tiempo a Vue para que prepare el componente antes de mostrarlo
  await nextTick()
  groupModalRef.value.show()
}

async function dupGroup(id) { //Se duplica con el MISMO nombre
  entityDup.value = M.resolve(id);
  console.log('duping group', id);
  await nextTick();
  dupModalRef.value.show();
}

async function rmGroup(id) {
  console.log("now deleting", id)
  vmOrGroupToDelete.value = (id == -1) ? [null, false] : [ M.resolve(id), false];
  // da tiempo a Vue para que prepare el componente antes de mostrarlo
  await nextTick()
  delModalRef.value.show()
  
  //M.rmGroup(id);
  //if (selected.value.id == id) {
  //  selected.value = {id: -1};
  //}
  //refresh();
}

function updateGroupMembers(id) {
  let members = M.resolve(id).members;
  for (let index = 0; index < members.length; index++) {
    let vm = M.resolve(members[index]);
    let hasGroup = false;
    for (let gr_index = 0; gr_index < vm.groups.length; gr_index++) {
      if (vm.groups[gr_index] == id) {
        hasGroup = true;
      }
    }

    if (!hasGroup) {
      vm.groups.push(id);
    }
  }
}

function removeGroupMembers(id) {
  let members = M.resolve(id).members;
  for (let index = 0; index < members.length; index++) {
    let vm = M.resolve(members[index]);
    for (let gr_index = 0; gr_index < vm.groups.length; gr_index++) {
      if (vm.groups[gr_index] == id) {
        vm.groups.splice(gr_index, 1);
      }
    }
  }
}

/////
// Búsqueda, Filtrado y Cambio de Estados
/////

const searchGroupQuery = ref({all: '', fields: []})
const searchVmQuery = ref({all: '', fields: []})
const debug = false;

const vmFilterGroup = ref(null)
const groupFilterVm = ref(null)

// muestra sólo vms de ese grupo (o todas con -1)
const switchVms = (groupId) => {
  console.log('vms for group ', groupId);
  vms.value = (groupId == -1) ? 
    M.getVms() :
    M.resolve(groupId).members.map(vmId => M.resolve(vmId))
  vmFilterGroup.value = (groupId == -1) ? 
    null :
    M.resolve(groupId)    
}

// muestra sólo grupos con esa vm (o todos con -1)
const switchGroups = (vmId) => {
  console.log('groups for vm ', vmId);
  groups.value = (vmId == -1) ? 
    M.getGroups() :
    M.resolve(vmId).groups.map(groupId => M.resolve(groupId))
  groupFilterVm.value = (vmId == -1) ? 
    null :
    M.resolve(vmId)        
}

// Editada y movida para el Ejercicio 7
// Ahora determina si está tratando con un grupo o con una vm, y cambia su estado
function setState(id, state) {
  const entity = M.resolve(id);
  if (Array.isArray(entity.groups)) {
    entity.state = state;
    M.setVm(entity);
  }
  else {
    entity.members.map(vmId => setState(vmId, state));
  }
  refresh();
}

function setManyStates(ids, state) {
  for (let index in ids) {
    console.log(ids);
    setState(ids[index], state);
  }
  refresh();
}

function manageCheckVms(id) {
  if (id != -1) {
    console.log("managing check...")
    let hasId = false;
    for (let index in checkboxed_vms) { 
      if (checkboxed_vms[index] == id) {
        hasId = true;
        console.log("check found, removing...")
        checkboxed_vms = checkboxed_vms.filter((elem) => elem != id);
      }
    } 
    if (!hasId) {
      console.log("check not found, adding...")
      checkboxed_vms.push(id);
    }
    console.log("result ", checkboxed_vms)
  } 
}

function manageCheckGroups(id) {
  if (id != -1) {
    console.log("managing check...")
    let hasId = false;
    for (let index in checkboxed_groups) { 
      if (checkboxed_groups[index] == id) {
        hasId = true;
        console.log("check found, removing...")
        checkboxed_groups = checkboxed_groups.filter((elem) => elem != id);
      }
    } 
    if (!hasId) {
      console.log("check not found, adding...")
      checkboxed_groups.push(id);
    }
    console.log("result ", checkboxed_groups)
  } 
}

</script>

<template>
  <!-- Navbar principal -->
  <nav class="navbar navbar-expand-lg">
    <div class="container-fluid">
      <a class="navbar-brand" href="#" title="Ir al inicio" >VManager</a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation" title="Menú">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="#div-groups" title="Ir a los grupos">Grupos</a>
        </li>
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="#div-vms" title="Ir a las máquinas virtuales">Vms</a>
        </li>
        </ul>
      </div>
    </div>
  </nav>

  <!-- 
    Div principal; container-fluid expande el contenedor para que ocupe todo el espacio disponible 
  -->
  <div class="container-fluid">
    <div class="row">
      <!-- 1a columna: grupos -->
      <div id="div-groups" class="col-md">
        <div>
          <h5 class="d-inline">Grupos
            <span v-if="groupFilterVm" class="filter"
              @click="switchGroups(-1)">
              que contienen a 
              <span class="name">{{ groupFilterVm.name }}×</span>
            </span>            
          </h5>
          <a class="d-inline d-sm-none details" href="#div-details" title="Ir a Detalles">↘️</a>
        </div>        
        <span v-if="debug"> {{ searchGroupQuery }}</span>      
        <FilterAddBox 
          v-model="searchGroupQuery" 
          :cols="['name', 'members']"
          :has-selected="checkboxed_groups.length > 0"
          @add-element="edGroup(-1)"
          addBtnTitle="Añadir nuevo grupo"
          @set-state="(state) => setManyStates(checkboxed_groups, state)"
          runBtnTitle="Iniciar un grupo"
          suspendBtnTitle="Suspender un grupo" 
          stopBtnTitle="Apagar un grupo" /> 

        <div class="overflow-y-scroll vh-100">
            <VmGrid :data="groups" :columns="['name', 'members']" :filter-key="searchGroupQuery.all"
            @choose="(e) => { console.log('selected group', e); selected = M.resolve(e) }"
            @checkboxed="(id) => manageCheckGroups(id)">
            </VmGrid>
        </div>
      </div>
      <!-- 2a columna: vms -->
      <div id="div-vms" class="col-md">
        <div>
          <a class="d-inline d-sm-none escape" href="#" title="Ir al inicio">⬆️</a>
          <h5 class="d-inline">Máquinas Virtuales 
            <span v-if="vmFilterGroup" class="filter"
              @click="switchVms(-1)">
              dentro de
              <span class="name">{{ vmFilterGroup.name }}×</span>
            </span>            
          </h5>
          <a class="d-inline d-sm-none details" href="#div-details" title="Ir a Detalles">↘️</a>
        </div>           
        <span v-if="debug"> {{ searchVmQuery }}</span>
        <FilterAddBox 
          v-model="searchVmQuery" 
          :cols="['name', 'ram', 'hd', 'ip']"
          :has-selected="checkboxed_vms.length > 0"
          @add-element="edVm(-1)"
          addBtnTitle="Añadir nueva VM"
          @set-state="state => setManyStates(checkboxed_vms, state)"
          runBtnTitle="Iniciar una VM"
          suspendBtnTitle="Suspender una VM" 
          stopBtnTitle="Apagar una VM" />
          
        <div class="overflow-y-scroll vh-100">
          <VmGrid :data="vms" :columns="['name', 'ram', 'groups', 'state']" :filter-key="searchVmQuery.all"
          @choose="(e) => { console.log('selected vm', e); selected = M.resolve(e) }"
          @checkboxed="(id) => manageCheckVms(id)">
          </VmGrid>
      </div>
      </div>
      <!-- 3a zona: detalles vms actuales -->
      <div id="div-details" class="col-md">
        <div>
          <a class="d-inline d-sm-none escape" href="#" title="Ir al inicio">⬆️</a>
          <h5 class="d-inline">Detalles</h5>
        </div>   
        <div id="details" class="container">
          <DetailsPane 
            :element="selected"
            @editVm="edVm(selected.id)"
            @dupVm="dupVm(selected.id)"
            @filterVm="switchGroups(selected.id)"
            @rmVm="rmVm(selected.id)"
            @editGroup="edGroup(selected.id)"
            @dupGroup="dupGroup(selected.id)"
            @filterGroup="switchVms(selected.id)"
            @rmGroup="rmGroup(selected.id)"
            @setState="state=>setState(selected.id, state)"
            @choose="(e) => { console.log('selected from details', e); selected = M.resolve(e) }"
          ></DetailsPane>
        </div>
      </div>
    </div>
  </div>

  <!-- 
    Modal para crear/editar grupo
    siempre usamos el mismo, y no se muestra hasta que hace falta
  -->
  <GroupAddOrEditModal ref="groupModalRef" 
    :key="groupToAddOrEdit.id"
    :group="groupToAddOrEdit" :isAdd="groupToAddOrEdit.id == -1"
    @add="(g) => { console.log('adding', g); let gr = M.addGroup(g); nextTick(); updateGroupMembers(gr.id); refresh() }"
    @edit="(g) => { console.log('setting', g); removeGroupMembers(g.id); M.setGroup(g); updateGroupMembers(g.id); refresh() }"
    />

  <!-- 
    Modal para crear/editar VM
    siempre usamos el mismo, y no se muestra hasta que hace falta
  -->
  <VmAddOrEditModal ref="vmModalRef"
    :key="vmToAddOrEdit.id"
    :vm="vmToAddOrEdit" :isAdd="vmToAddOrEdit.id == -1"
    @add="(vm) => { console.log('adding', vm); let v = M.addVm(vm); nextTick(); refresh(); updateVmGroups(v.id); refresh() }"
    @edit="(vm) => { console.log('setting', vm); removeVmGroups(vm.id); M.setVm(vm); updateVmGroups(vm.id); refresh() }"
    />

    <!-- 
    Modal para duplicar VMs y grupos
    siempre usamos el mismo, y no se muestra hasta que hace falta
  -->
  <DupModal ref="dupModalRef"
    :key="entityDup.id"
    :entity="entityDup" :exists="entityDup.id != -1" :isVm="exists && Array.isArray(M.resolve(entityDup.id).groups)"
    @dupVm="(vm) => { console.log('duping', vm); let v = M.addVm(vm); nextTick(); refresh(); updateVmGroups(v.id); refresh() }"
    @dupGroup="(group) => { console.log('duping', group); let gr = M.addGroup(group); nextTick(); refresh(); updateGroupMembers(gr.id); refresh() }"
    />

  <!-- 
    Modal para borrar VMs y grupos
    siempre usamos el mismo, y no se muestra hasta que hace falta
  -->
  <DeletingModal ref="delModalRef"
    :key="vmOrGroupToDelete.at(0).id"
    :vmOrg="vmOrGroupToDelete.at(0)" :isVM="vmOrGroupToDelete.at(1)"
    @dlVm="(id) => { console.log('deleting VM', id); removeVmGroups(id); M.rmVm(id); if (selected.id == id) {selected = {id: -1};} refresh() }"
    @dlGroup="(id) => { console.log('deleting Group', id); removeGroupMembers(id); M.rmGroup(id); if (selected.id == id) {selected = {id: -1};} refresh() }"
  />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
.escape {
  font-size: 150%;
  padding-right: 1em;
}
.details {
  font-size: 150%;
  padding-left: 1em;
}
span.filter>.name {
  font-weight: 1000;
}
span.filter {
  border-bottom: 2px dashed gray;
}
span.filter:hover {
  border-bottom: 2px dashed blue;
}

</style>
