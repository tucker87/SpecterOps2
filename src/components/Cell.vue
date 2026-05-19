<script setup>
import { toCoord, dice } from '~/utils'

const cell = defineModel()
const emit = defineEmits(['dropPlayer'])

const allowsMovement = cell.value.isOpen || cell.value.isStreet || cell.value.isCrossroad || cell.value.isExit
const showCoords = cell.value.isOpen || cell.value.isStreet || cell.value.isCrossroad
const handleDrop = e => {
   if (allowsMovement)
      emit('dropPlayer', e)
}

const isGoal = cell.value.code < 92
const diceSide = isGoal && dice[(cell.value.code - 65) % 6]
const goalIndex = isGoal && Math.floor((cell.value.code - 65) / 6) + 1
</script>

<template>
   <div class="cell" :class="[cell.type, cell.facing, cell.active ? 'active' : '', cell.seen ? 'seen' : '']"
      :style="cell.style" @drop="handleDrop" @dragover.prevent @dragenter.prevent>
      <template v-if="showCoords">
         {{ toCoord(cell) }}{{ cell.y }}
      </template>

      <template v-else-if="isGoal">
         <span class="index">{{ goalIndex }}</span>
         <span class="die">{{ diceSide }}</span>
      </template>

      <template v-if="cell.type === 'e'">
         <span>→</span>
      </template>

      <template v-if="cell.type === 'c'">
         <span>N</span>
         <span>E</span>
         <span>S</span>
         <span>W</span>
      </template>
   </div>
</template>
