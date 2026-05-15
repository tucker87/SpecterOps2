<script setup>
import { ref } from 'vue';
import { baseBoard } from '~/boards.js';

const boardData = baseBoard
const boardWidth = 23
const boardHeight = 32

const selectableCellTypes = ['e', 'o', 's', 'x', 'y']

const cellIndex = (x, y) => {
   if (x > boardWidth || x < 1)
      return

   return (y - 1) * boardWidth + x - 1

}
const range = (start, end) => Array.from({ length: end - start + 1 }, (_, i) => start + i);

let board = ref(boardData
   .replaceAll('\n', '')
   .trim()
   .split('')
   .map((type, i) => {
      const x = 1 + i % boardWidth
      const y = 1 + Math.floor(i / boardWidth)
      const isGoal = type >= 'A' && type <= 'Z'

      return {
         type,
         x,
         y,
         code: type.charCodeAt(),
         blocksVision: type === 'w' || type === 'c' || isGoal,
         selectable: selectableCellTypes.some(valid => type === valid),
         isOpen: type === 'o',
         isStreet: type === 's',
         isCrossroad: type === 'x' || type === 'y',
         isGoal,
         neighborStreets: [],
      }
   }))

const findStreetNeighbors = (board) => {
   for (var c of board) {
      const { x, y } = c

      const fourNeighbors = {
         n: board[cellIndex(x, y - 1)],
         e: board[cellIndex(x + 1, y)],
         s: board[cellIndex(x, y + 1)],
         w: board[cellIndex(x - 1, y)],
      }

      const facing = Object.keys(fourNeighbors).filter(d => !fourNeighbors[d])[0]
      if (facing)
         c.facing = 'f-' + facing

      const singleStreet = (a, b) => {
         if (!a || !b) return
         if (a.isStreet !== b.isStreet)
            return a.isStreet ? a : b
      }

      if (c.isCrossroad) {
         for (var key in fourNeighbors) {
            const n = fourNeighbors[key]
            if (n.isCrossroad && n.type === c.type)
               c.neighborStreets.push(n)
         }
      } else {
         const a = singleStreet(fourNeighbors.n, fourNeighbors.s)
         const b = singleStreet(fourNeighbors.e, fourNeighbors.w)

         if (a)
            c.neighborStreets.push(a)
         if (b)
            c.neighborStreets.push(b)
      }
   }
}

findStreetNeighbors(board.value)

// const players = ref([{ name: 'one', x: 2, y: 2 }, { name: 'two', x: 3, y: 3 }])

const moves = ref([])
const toCoord = pos => String.fromCharCode(pos.x + 64)
const dice = ['⚀', '⚁', '⚂', '⚃', '⚄', '⚅']

const selectCell = cell => {
   board.value.forEach(cell => cell.active = false)

   if (cell.active || !cell.selectable) {
      clearSeen()
      return
   }

   cell.active = true
   clearSeen()
   updateSeen(cell)
}

const clearSeen = () => board.value.forEach(cell => cell.seen = false)

const updateSeen = activeCell => {
   if (!activeCell)
      return;

   let { x, y } = activeCell

   const left = range(1, x - 1).map(cx => cellIndex(cx, y)).reverse()
   const right = range(x + 1, boardWidth).map(cx => cellIndex(cx, y))
   const up = range(1, y - 1).map(cy => cellIndex(x, cy)).reverse()
   const down = range(y + 1, boardHeight).map(cy => cellIndex(x, cy))

   const onRoad = activeCell.isStreet || activeCell.isCrossroad

   setSeen(left, onRoad)
   setSeen(right, onRoad);
   setSeen(up, onRoad);
   setSeen(down, onRoad);
}

const setSeen = (cellIndexes, onRoad) => {
   for (const cx of cellIndexes) {
      const cell = board.value[cx]
      if (cell.blocksVision)
         break

      cell.seen = true
      if (onRoad)
         for (const n of cell.neighborStreets)
            n.seen = true
   }
}

const addMove = cell => moves.value.push({ pos: `${toCoord(cell)}${cell.y}` })
const deleteMove = cell => moves.value = moves.value.filter(x => x !== cell)

</script>

<template>
   <section>
      <div>
         <h1>Specter Ops</h1>
         <p>Original / Broken Covenant</p>
      </div>
   </section>
   <div style="display: grid; grid-template-columns: 1fr 100px;">
      <section id="board">
         <template v-for="cell in board" :style="`grid-row: ${cell.x}; grid-column: ${cell.y};`">
            <div class="cell" :class="[cell.type, cell.facing, cell.active ? 'active' : '', cell.seen ? 'seen' : '']"
               @click="selectCell(cell)" @dblclick="addMove(cell)">
               <template v-if="cell.isOpen || cell.isStreet || cell.isCrossroad">
                  {{ toCoord(cell) }}{{ cell.y }}
               </template>

               <template v-else-if="cell.code < 92">
                  <span class="index">{{ ((cell.code - 65) % 6) + 1 }}</span>
                  <span class="die">{{ dice[(cell.code - 65) % 6] }}</span>
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
         <div class="player" v-for="player in players" :style="`grid-row: ${player.x}; grid-column: ${player.y};`">
            {{ player.name }}
         </div>
      </section>

      <section id="moves">
         <template v-for="(move, i) in moves">
            <div class="move" @click="deleteMove(move)">{{ i + 1 }}. {{ move.pos }}</div>
         </template>
      </section>
   </div>
   <section id="spacer"></section>
</template>

<!-- Theme -->
<style>
body {
   --primary-color: #9ca3af;
   --open-background: #FFF;
   --street-background: #EEE;
   --wall-background: #444;
   --title-background: #444;
   --title-color: #FFF;
   --active-tint: blue 40%;
   --active-color: #000;
   --seen-tint: blue 20%;
   --goal-background: #444;
   --goal-index-color: #FFF;
   --goal-die-color: #00AA00;
}
</style>

<style lang="scss">
#moves {
   position: sticky;
   top: 0;
   align-self: start;
   cursor: pointer;
}

#board {
   color: var(--primary-color);
   font-size: 12px;
   user-select: none;
   cursor: pointer;

   display: grid;
   grid-template-columns: repeat(23, 1fr);
   grid-gap: 1px;

   .cell {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      aspect-ratio: 1;

      &.active {
         color: var(--active-color);
      }

      &.w {
         background-color: var(--wall-background);

         &.active {
            background-color: color-mix(in srgb, var(--wall-background), var(--active-backgound))
         }
      }

      &.o,
      &.e {
         background-color: var(--open-background);

         &.active {
            background-color: color-mix(in srgb, var(--open-background), var(--active-tint))
         }

         &.seen {
            background-color: color-mix(in srgb, var(--open-background), var(--seen-tint))
         }
      }

      &.e.f-n {
         rotate: -90deg;
      }

      &.e.f-w {
         rotate: 180deg;
      }

      &.e.f-s {
         rotate: 90deg;
      }

      &.s,
      &.x,
      &.y {
         background-color: var(--street-background);

         &.active {
            background-color: color-mix(in srgb, var(--street-background), var(--active-tint))
         }

         &.seen {
            background-color: color-mix(in srgb, var(--street-background), var(--seen-tint))
         }
      }

      &.t {
         aspect-ratio: unset;
         grid-column: span 5;
         color: var(--title-color);
         background: var(--title-background);
         font-size: clamp(0rem, 5vw, 1rem);

         &::before {
            content: 'FACILITY 83X';
         }
      }

      &.t~.t,
      &.c~.c {
         display: none;
      }

      &.c {
         aspect-ratio: unset;
         background-color: #0000AA;
         grid-column: span 2;
         grid-row: span 2;
         position: relative;

         span {
            position: absolute;
         }

         span:nth-child(1) {
            top: 5px;
         }

         span:nth-child(2) {
            right: 10px;
         }

         span:nth-child(3) {
            bottom: 5px;
         }

         span:nth-child(4) {
            left: 10px;
         }
      }


      $alphabet: A B C D E F G H I J K L M N O P Q R S T U V W X Y Z;

      @each $letter in $alphabet {
         &.#{$letter} {
            background: var(--goal-background);
            display: flex;
            justify-content: center;
            align-items: center;
            color: var(--goal-die-color);
            font-size: 30px;

            .index {
               color: var(--goal-index-color);
               font-size: initial;
            }
         }
      }
   }

   .player {
      width: 100%;
      aspect-ratio: unset;
   }
}
</style>
