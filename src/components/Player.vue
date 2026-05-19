<script setup>
import { computed } from 'vue'

const emit = defineEmits(['moveToCar'])

const player = defineModel()
const dragStart = (e, id) => {
   e.dataTransfer.setData('playerId', id);
}
const style = computed(() => player.value.inCar ? '' : `grid-area: ${player.value.y} / ${player.value.x};`)
</script>

<template>
   <div class="player" :style="style" draggable="true" @dragstart="dragStart($event, player.id)"
      @dblclick="emit('moveToCar')">
      {{ player.name }}
   </div>
</template>
