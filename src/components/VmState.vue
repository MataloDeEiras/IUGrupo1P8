<script setup>

  import { VmState } from '../model.js'

  defineProps({
    state: String
  })

  const vmStateIcons = {
    [VmState.RUNNING]: "▶",
    [VmState.SUSPENDED]: "💤",
    [VmState.STOPPED]: "🛑"
  };

  const vmStateClasses = {
    [VmState.RUNNING]: "badge rounded-pill text-bg-success",
    [VmState.SUSPENDED]: "badge rounded-pill text-bg-warning opacity-50",
    [VmState.STOPPED]: "fix-stop badge rounded-pill text-bg-danger"
  };  
</script>

<template>
  <span :class="vmStateClasses[state]" class="align-items-center" :title="state">{{vmStateIcons[state]}}</span>
</template>

<style scoped>
span { height: 2em; width: 3em; font-size: 70% }
</style>
