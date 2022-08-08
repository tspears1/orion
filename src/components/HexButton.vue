<template>
   <div class="hexButton">
      <div class="hexButton__grid">
         <Presence
            v-for="index in compass.length"
            :key="index"
         >
         <Motion
            class="hexButton__cell"
            v-if="isOpen"
            :data-index="index"
            :initial="closedMotion(index)"
            :animate="openedMotion(index)"
            :exit="closedMotion(index)"
         />
         </Presence>
      </div>
      <button
         class="hexButton__trigger hexButton__cell"
         ref="trigger"
         @mouseover="openHex"
         @mouseleave="closeHex"
         @focus="openHex"
         @blur="closeHex"
      >
         <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
            <path d="M509.7 261.7c3.125-3.125 3.125-8.188 0-11.31l-152-152C356.1 96.78 354.1 95.1 352 95.1s-4.094 .7813-5.656 2.344c-3.125 3.125-3.125 8.188 0 11.31l138.3 138.3H8c-4.406 0-8 3.578-8 8C0 260.4 3.594 263.1 8 263.1h476.7l-138.3 138.3c-3.125 3.125-3.125 8.188 0 11.31s8.188 3.125 11.31 0L509.7 261.7z"/>
         </svg>
      </button>
   </div>
</template>

<script setup>
   import { ref, reactive, onMounted, watch } from 'vue'
   import { Motion, Presence } from 'motion/vue'
   import { animate, spring } from 'motion'

   // Utility ================================
   const compass = ['s', 'se', 'ne', 'n', 'nw', 'sw']
   const hexStart = {
      s: {
         x: 0,
         y: -2,
         origin: 'top',
      },
      se: {
         x: -1,
         y: -1,
         origin: 'top left',
      },
      ne: {
         x: -1,
         y: 1,
         origin: 'bottom left',
      },
      n: {
         x: 0,
         y: 2,
         origin: 'bottom',
      },
      nw: {
         x: 1,
         y: 1,
         origin: 'bottom right',
      },
      sw: {
         x: 1,
         y: -1,
         origin: 'top right',
      },
   }

   const closedMotion = (index) => {
      const point = hexStart[`${compass[index - 1]}`]
      const moveX = ( grid.columnSize + grid.columnGap ) * point.x
      const moveY = ( grid.rowSize + grid.rowGap ) * point.y

      return {
         opacity: 0,
         rotate: 360,
         scale: 0,
         x: moveX,
         y: moveY,
         transformOrigin: point.origin,
         transition: closeTransition(index),
      }
   }

   const openedMotion = (index) => {
      return {
         opacity: 0.85,
         rotate: 0,
         scale: 1,
         x: 0,
         y: 0,
         transition: openTransition(index),
      }
   }

   const reverseDelay = (index) => compass.length - index + 1

   const openTransition = (index) => {
      return {
         delay: index * 0.05,
         duration: 0.5,
         x: {
            easing: spring(),
         },
         y: {
            easing: spring(),
         },
         scale: {
            easing: spring(),
         }
      }
   }

   const closeTransition = (index) => {
      return {
         delay: reverseDelay(index) * 0.025,
      }
   }

   // Data =====================================
   const isOpen = ref(false)
   const trigger = ref({})
   const grid = reactive({ columnGap: 0, columnSize: 0, rowGap: 0, rowSize: 0 })

   // Lifecycle ================================
   onMounted(() => {
      updateGridVariables()
   })

   watch(isOpen, (value) => {
      if (value) {
         animate(trigger.value,{ scale: [null, 0.8, 2, 1], rotate: -360 },
            { easing: spring({ stiffness: 35 }) }).finished.then(() => {
               trigger.value.style = null
            })
      }
   })

   // Methods ==================================
   const updateGridVariables = () => {
      grid.columnSize = parseInt(getComputedStyle(document.documentElement).getPropertyValue('--hex-column-size'), 10),
      grid.rowSize = grid.columnSize / 1.5
      grid.columnGap = grid.columnSize / 2.727272
      grid.rowGap = grid.rowSize / 5
   }

   const toggleHex = () => {
      isOpen.value = !isOpen.value
   }

   const closeHex = () => {
      isOpen.value = false
   }

   const openHex = () => {
      isOpen.value = true
   }

</script>

<style lang="scss">
:root {
   --hex-column-size: 60px;
   --hex-row-size: calc(var(--hex-column-size) / 1.5);
   --hex-column-gap: calc(var(--hex-column-size) / 2.727272);
   --hex-row-gap: calc(var(--hex-row-size) / 5);
   --hex-cell-x: calc(var(--hex-column-gap) * 2 + var(--hex-column-size));
   --hex-cell-y: calc(var(--hex-row-size) * 2 + var(--hex-row-gap));
   --hex-grid-columns: 3;
   --hex-grid-rows: 6;

   // utility
   --hex-grid-y: calc(
    (var(--hex-grid-rows) * var(--hex-row-size)) +
      ((var(--hex-grid-rows) - 1) * var(--hex-row-gap))
   );
   --hex-grid-x: calc(
      (var(--hex-grid-columns) * var(--hex-column-y)) +
         ((var(--hex-grid-columns) + 1) * var(--hex-column-gap))
   );
}

.hexButton {
   $b: &;
   position: relative;

   &__trigger {
      border: 0;
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      grid-template-rows: repeat(3, 1fr);
      margin: 0;
      padding: 0;

      &::after {
         content: '';
         inset: 0;
         position: absolute;
      }

      &:hover,
      &:focus {
         background: #464a9b;
         cursor: pointer;
      }

      svg {
         fill: #fff;
         grid-column: 2;
         grid-row: 2;
         height: 100%;
         object-fit: contain;
         stroke-width: 3%;
         stroke: #fff;
         width: 100%;
      }
   }

   &__grid {
      display: grid;
      gap: var(--hex-row-gap) var(--hex-column-gap);
      grid-template-columns: auto repeat(var(--hex-grid-columns), var(--hex-column-size)) auto;
      grid-template-rows: repeat(var(--hex-grid-rows), var(--hex-row-size));
      justify-items: center;
      place-content: center;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
   }

   &__cell {
      background: #646cff;
      clip-path: polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0% 50%);
      height: var(--hex-cell-y);
      width: var(--hex-cell-x);

      &[data-index] {
         grid-column: span 3;
         grid-row: span 2;
      }
      &[data-index="1"] {
         grid-column-start: 3;
         grid-row-start: 5;
      }
      &[data-index="2"] {
         grid-column-start: 4;
         grid-row-start: 4;
      }
      &[data-index="3"] {
         grid-column-start: 4;
         grid-row-start: 2;
      }
      &[data-index="4"] {
         grid-column-start: 3;
         grid-row-start: 1;
      }
      &[data-index="5"] {
         grid-column-start: 2;
         grid-row-start: 2;
      }
      &[data-index="6"] {
         grid-column-start: 2;
         grid-row-start: 4;
      }
   }
}
</style>