<script setup>
import { ref } from 'vue';
import { baseBoard, bcBoard } from '~/boards.js';
import Cell from './Cell.vue'
import Player from './Player.vue'

// const boardData = baseBoard
const boardData = bcBoard
const boardWidth = 23
const boardHeight = 32

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
      const style = type !== 't' && type !== 'c' ? `grid-area: ${y} / ${x};` : ''

      return {
         type,
         x,
         y,
         code: type.charCodeAt(),
         blocksVision: type === 'w' || type === 'c' || isGoal,
         isOpen: type === 'o',
         isStreet: type === 's',
         isCrossroad: type === 'x' || type === 'y',
         isExit: type === 'e',
         isGoal,
         neighborStreets: [],
         style,
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

const car = ref({ x: 11, y: 23 }) // 2: K17, 3: K23
const initPlayer = index => ({
   id: index,
   name: '' + index,
   x: car.value.x,
   y: car.value.y,
   inCar: true,
})

const players = ref([
   initPlayer(1),
   initPlayer(2),
   initPlayer(3),
   initPlayer(4),
])

const moves = ref([])
const toCoord = pos => String.fromCharCode(pos.x + 64)

const clearSeen = () => board.value.forEach(cell => cell.seen = false)

const updatePlayerSeen = () => {
   for (const player of players.value) {
      const { x, y } = player

      const left = range(1, x - 1).map(cx => cellIndex(cx, y)).reverse()
      const right = range(x + 1, boardWidth).map(cx => cellIndex(cx, y))
      const up = range(1, y - 1).map(cy => cellIndex(x, cy)).reverse()
      const down = range(y + 1, boardHeight).map(cy => cellIndex(x, cy))

      const activeCell = board.value[cellIndex(x, y)]
      if (!activeCell)
         return

      activeCell.seen = true

      const onRoad = activeCell.isStreet || activeCell.isCrossroad

      setSeen(left, onRoad)
      setSeen(right, onRoad);
      setSeen(up, onRoad);
      setSeen(down, onRoad);
   }
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

const dropPlayer = (e, x, y) => {
   e.preventDefault();
   const playerId = e.dataTransfer.getData('playerId');

   if (playerId === 'car') {
      car.value.x = x
      car.value.y = y
      for (const player of players.value) {
         if (player.inCar) {
            player.x = x
            player.y = y
         }
      }
   }
   else {
      const player = players.value.find(p => p.id === +playerId)
      player.inCar = false
      player.x = x
      player.y = y
   }

   clearSeen()
   updatePlayerSeen()
}

const moveToCar = player => {
   player.x = car.value.x
   player.y = car.value.y
   player.inCar = true

   clearSeen()
   updatePlayerSeen()
}

const dragCar = e => {
   e.dataTransfer.setData('playerId', 'car');
}

updatePlayerSeen()
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
         <cell v-for="(cell, i) in board" v-model="board[i]" @dropPlayer="dropPlayer($event, cell.x, cell.y)"
            @dblclick="addMove(cell)" />

         <div class="car" :style="`grid-area: ${car.y} / ${car.x};`" draggable="true" @dragstart="dragCar">
            CAR
         </div>

         <template v-for="(player, i) in players">
            <player v-if="!player.inCar" v-model="players[i]" @moveToCar="moveToCar(player)" />
         </template>
      </section>

      <section id="sidebar">
         <div class="insideCar"><span>In Car</span>
            <div class="carGrid">
               <template v-for="i in 4">
                  <div class="cell">
                     <template v-if="players[i - 1]">
                        <player v-if="players[i - 1].inCar" v-model="players[i - 1]" />
                        <div v-else>-</div>
                     </template>
                  </div>
               </template>

            </div>
         </div>
         <span>Moves</span>
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
   --seen-tint: blue 20%;
   --goal-background: #444;
   --goal-index-color: #FFF;
   --goal-die-color: #00AA00;
}
</style>

<style lang="scss">
#sidebar {
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

      &.w {
         background-color: var(--wall-background);
      }

      &.o,
      &.e {
         background-color: var(--open-background);

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


}

.player {
   display: flex;
   justify-content: center;
   align-items: center;
   background-color: #AA0000;
   border-radius: 100%;
   z-index: 10;
   aspect-ratio: 1;
}

.car {
   display: flex;
   justify-content: center;
   align-items: center;
   background-color: #AA0000;
   border-radius: 100%;
   z-index: 10;
}

.insideCar {
   display: flex;
   justify-content: space-around;
   align-items: center;
   flex-direction: column;
   margin: 5px;
}

.carGrid {
   display: grid;
   grid-gap: 1px;
   width: 100%;
   aspect-ratio: 1;
   grid-template-columns: 1fr 1fr;
   grid-template-rows: 1fr 1fr;
   border: 1px solid blue;

   background-color: gray;

   .cell {
      background-color: white;
   }
}
</style>
