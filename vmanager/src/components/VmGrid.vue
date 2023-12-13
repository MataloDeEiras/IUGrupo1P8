<script setup>
import { ref, computed } from 'vue'
import { resolve } from '../model.js'
import VmState  from './VmState.vue'

// basado en https://vuejs.org/examples/#grid

const props = defineProps({
  data: Array,
  columns: Array,
  filterKey: String
})
defineEmits(['choose', 'checkboxed'])

const sortKey = ref('')
const sortOrders = ref(
  props.columns.reduce((o, key) => ((o[key] = 1), o), {})
)

const filteredData = computed(() => {
  let { data, filterKey } = props
  if (filterKey) {
    filterKey = filterKey.toLowerCase()
    data = data.filter((row) => {
      return Object.keys(row).some((key) => {
        return String(row[key]).toLowerCase().indexOf(filterKey) > -1
      })
    })
  }
  const key = sortKey.value
  if (key) {
    const order = sortOrders.value[key]
    data = data.slice().sort((a, b) => {
      a = a[key]
      b = b[key]
      return (a === b ? 0 : a > b ? 1 : -1) * order
    })
  }
  return data
})

function sortBy(key) {
  sortKey.value = key
  sortOrders.value[key] *= -1
}

function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1)
}
  
function head(array, n) {
  return array.slice(0,n);  
}

function tail(array, n) {
  return array.slice(n);
}

function pretty(array) {
   return array.map(v => resolve(v).name).join(", ")
}
</script>

<template>
  <table class="table table-hover" v-if="filteredData.length">
    <thead>
      <tr>
        <th v-for="key in columns" :key="key"
          @click="sortBy(key)"
          :class="{ active: sortKey == key }">
          {{ capitalize(key) }}
          <span v-if="sortKey==key" :class="`arrow ${sortOrders[key] > 0 ? 'asc' : 'dsc'}`">
          </span>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="entry in filteredData" :key="entry.id"
        @click="$emit('choose', entry.id)">
        
        <td v-for="key in columns" :key="`_${entry.id}_${key}`" class="text-start">
          
          <template v-if="key === 'name'">
            <input class="form-check-input" type="checkbox" @click="$emit('checkboxed', entry.id)">
            <span class="name px-3">{{entry[key]}}</span>
          </template>
          <template v-else-if="key === 'ram'">
            {{entry[key]}}{{" Gb"}}
          </template>
          <template v-else-if="Array.isArray(entry[key])">
            <span v-if="(entry[key].length == 0)" class="less-relevant">------</span>
            <span v-if="(entry[key].length > 3)">
              <template v-for="item in head(entry[key], 3)" :key="item">
                <span class="badge text-bg-light">{{resolve(item).name}}</span>
              </template>
            </span>
            <span class="badge text-bg-light" :title="pretty(tail(entry[key], 3))" v-if="(entry[key].length > 3)">  +{{tail(entry[key], 3).length}}  </span> 
            <p v-else>
              <template v-for="item in entry[key]" :key="item">
                <span class="badge text-bg-light">{{resolve(item).name}}</span>
              </template>
            </p> 
          </template>
          
          <template v-else-if="key === 'state'">
            <VmState :state="entry[key]">
            </VmState>
          </template>
          <template v-else>
            {{entry[key]}}
          </template>
        </td>
      </tr>
    </tbody>
  </table>
  <p v-else>(no hay nada que mostrar)</p>
</template>

<style>
  .table>thead>tr>th {
    background-color: white;
    position: sticky;
    position: -webkit-sticky;
    top: 0;
    z-index: 2;
  }
  span.badge.text-bg-light {
    margin-left: 3px;
    margin-right: 3px;
  }
  .arrow.asc::after {
    content: "↓"
  }
  .arrow.dsc::after {
    content: "↑"
  }
  span.name {
    font-weight: 1000;
  }
  table {
    margin-top: 10px;

  }
  thead>tr {
    border-bottom: 1px solid gray;
    color: rgb(104, 103, 103);
  }
  .less-relevant{
    color: #787878
  }
</style>
