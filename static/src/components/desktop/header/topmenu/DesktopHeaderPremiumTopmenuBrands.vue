<template>
  <div :class="$style.root">
    <x-link
      v-for="item in brands"
      :key="item.id"
      :class="$style.item"
      :href="item.link || '/'"
    >
      <div :class="$style.background" :style="{ backgroundImage: `url(${item.image})` }"></div>
    </x-link>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { useDevice } from 'composables/useDevice';

import XLink from 'blocks/x-link/x-link.vue';

const device = useDevice();

const props = defineProps({
  brandsList: {
    type: Array,
    required: true,
  },
});

const brands = computed(() => props.brandsList.filter(item => item.image).slice(0, device.isWide ? 10 : 8));
</script>

<style module>
.root {
  padding-top: 44px;
}

.item {
  display: inline-flex;
  align-items: center;
  width: 98px;
  height: 82px;
  position: relative;
}

.item:not(:first-child) {
  margin-left: 24px;
}

.background {
  background-repeat: no-repeat;
  background-position: center;
  background-size: contain;
  height: 100%;
  width: 100%;
  transition: opacity var(--animationDuration) ease-out;
  opacity: 0.546;
}

.item:hover .background {
  opacity: 1;
}
</style>
