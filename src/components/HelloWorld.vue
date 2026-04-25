<script setup>
import { ref } from 'vue';

const boardData = `
oowowowowowooeoooottttt
wooooooooowowoowwowowwo
eowwowowwooowoowooooooe
wowooooowowoooooowowoww
oooowowooooowoowooooooo
owwowowwowowwoooowwowwo
ooooooooooooooowowooooo
ooooooooooooooooooowwow
wowwowowwowowoowowooooo
ooowooowooooooooooowowo
owoooEoooccowoowwowwowo
owowoooFoccoooooooooooo
ooooowoooooowoooooooooo
woDowwwowwowwoowwowoXow
oooooooooooooooooowoooo
ooooooooooooooowwowwoWo
woBowowowoowowooooooooo
oooooooowoowoooSoToUoVo
oAoCowwoooooowooooooooo
oooooowowoowowwowowowow
wowowoooooooooooooooooo
wowowwowwoooooooooooooo
ooooooooooowowowowwowwo
ooooooooooowoooooowoooo
owwowwowwoooowwoNoooQow
oowoooooooowoowooowoooo
woooIoJowoowwooowowwowo
woHooooowooooowoooooowo
oooowoKoooowowwoOowwooo
oGowwoooLoowoooooooooRo
oooooowooooooMowoPowooo
owwowowowoowooowooowowo
`

const boardWidth = 23
const boardHeight = 32

const board = ref(boardData
   .replaceAll('\n', '')
   .trim()
   .split('')
   .map((c, i) => ({
      i,
      type: c,
      x: 1 + i % boardWidth,
      y: 1 + Math.floor(i / boardWidth),
      code: c.charCodeAt()
   })))

const moves = ref([])
const toCoord = pos => String.fromCharCode(pos.x + 64)
const dice = ['⚀', '⚁', '⚂', '⚃', '⚄', '⚅']

const selectCell = cell => {
   board.value.forEach(cell => cell.active = false)

   if (cell.active || ['e', 'o'].every(valid => cell.type !== valid)) {
      updateSeen()
      return
   }

   cell.active = true
   updateSeen(cell)
}

const cellIndex = (x, y) => (y - 1) * boardWidth + x - 1
const range = (start, end) => Array.from({ length: end - start + 1 }, (_, i) => start + i);
const blocksVision = cell => cell.type === 'w' || cell.type === 'c'

const checkSeen = cellIndexes => {
   for (const cx of cellIndexes) {
      const cell = board.value[cx]
      if (blocksVision(cell))
         break

      cell.seen = true
   }
}

const updateSeen = activeCell => {
   board.value.forEach(cell => cell.seen = false)

   if (!activeCell)
      return;

   let { x, y } = activeCell

   const left = range(1, x - 1).map(cx => cellIndex(cx, y)).reverse()
   const right = range(x + 1, boardWidth).map(cx => cellIndex(cx, y))
   const up = range(1, y - 1).map(cy => cellIndex(x, cy)).reverse()
   const down = range(y + 1, boardHeight).map(cy => cellIndex(x, cy))

   checkSeen(left)
   checkSeen(right);
   checkSeen(up);
   checkSeen(down);
}

const addMove = cell =>
   moves.value.push({ pos: `${toCoord(cell)}${cell.y}` })

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
         <template v-for="cell in board">
            <div class="cell" :class="[cell.type, cell.active ? 'active' : '', cell.seen ? 'seen' : '']"
               @click="selectCell(cell)" @dblclick="addMove(cell)">
               <template v-if="cell.type === 'o'">
                  {{ toCoord(cell) }}{{ cell.y }}
               </template>

               <template v-else-if="cell.code < 92">
                  <span class="index">{{ ((cell.code - 65) % 6) + 1 }}</span>
                  <span class="die">{{ dice[(cell.code - 65) % 6] }}</span>
               </template>
            </div>
         </template>
      </section>

      <section id="moves">
         <div v-for="(move, i) in moves">
            <div class="move">{{ i + 1 }}. {{ move.pos }}</div>
         </div>
      </section>
   </div>
   <section id="spacer"></section>
</template>

<!-- Theme -->
<style>
body {
   --primary-color: #9ca3af;
   --open-background: #FFF;
   --wall-background: #444;
   --title-background: #444;
   --active-tint: blue 40%;
   --active-color: #000;
   --seen-tint: blue 20%;
   --goal-background: #444;
   --goal-index-color: #FFF;
   --goal-die-color: #00AA00;
}
</style>

<style lang="scss">
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

      &.o {
         background-color: var(--open-background);

         &.active {
            background-color: color-mix(in srgb, var(--open-background), var(--active-tint))
         }

         &.seen {
            background-color: color-mix(in srgb, var(--open-background), var(--seen-tint))
         }
      }

      &.t {
         aspect-ratio: unset;
         grid-column: span 5;
         color: #FFF;
         background: var(--title-background);
         font-size: clamp(0rem, 5vw, 1rem);

         &::before {
            content: 'FACILITY 83X';
         }
      }

      &.t~.t {
         display: none;
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
</style>
