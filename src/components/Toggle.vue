<script setup>
const emit = defineEmits(['change'])
const checked = defineModel({ type: Boolean, required: true })
</script>

<template>
   <label>
      <slot default name="before" />
   </label>
   <label class="switch">
      <input type="checkbox" :checked="checked" @change="checked = !checked; emit('change')" />
      <span class="slider"></span>
   </label>
   <label>
      <slot name="after" />
   </label>
</template>

<style>
.switch {
   --switch-width: 35px;
   --switch-height: 20px;
   --slider-width: 12px;
   --slider-move: calc(var(--switch-width) - var(--slider-width) * 1.5);

   position: relative;
   display: inline-block;
   width: var(--switch-width);
   height: var(--switch-height);
}

.switch input {
   opacity: 0;
   width: 0;
   height: 0;
}

.slider {
   position: absolute;
   cursor: pointer;
   inset: 0;
   background-color: #ccc;
   -webkit-transition: .4s;
   transition: .4s;
   border-radius: 100vw;
}

.slider:before {
   position: absolute;
   content: "";
   width: var(--slider-width);
   aspect-ratio: 1;
   left: 4px;
   bottom: 4px;
   background-color: white;
   -webkit-transition: .4s;
   transition: .4s;
   border-radius: 100%;
}

input:checked+.slider {
   background-color: #2196F3;
}

input:focus+.slider {
   box-shadow: 0 0 1px #2196F3;
}

input:checked+.slider:before {
   transform: translateX(var(--slider-move));
}
</style>
