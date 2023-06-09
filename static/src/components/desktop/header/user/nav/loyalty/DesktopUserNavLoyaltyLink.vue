<template>
  <x-link
    :href="href"
    :class="$style.root"
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <div v-if="counter" :class="$style.counter">{{ counter }}</div>
    <x-icon
      :type="isHovered ? activeIcon : icon"
      :class="$style.icon"
      size="32"
    />
    <slot ></slot>
  </x-link>
</template>

<script setup lang="ts">
import { ref } from 'vue';

import XLink from 'blocks/x-link/x-link.vue';
import XIcon from 'blocks/x-icon/x-icon.vue';

defineProps({
  icon: {
    type: String,
    required: true,
  },
  activeIcon: {
    type: String,
    required: true,
  },
  href: {
    type: String,
    required: true,
  },
  counter: {
    type: Number,
    default: 0,
  },
});

const isHovered = ref(false);
</script>

<style module>
.root {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 80px;
  margin: 0 20px;
  font: var(--fontSize);
  color: var(--labelColor);
  transition: none;
}

.root:first-child {
  margin-left: 0;
}

.root:last-child {
  margin-right: 0;
}

.root:hover {
  color: var(--secondaryLabelColor);
}

.icon {
  margin-bottom: 8px;
}

.counter {
  position: absolute;
  top: 0;
  right: 12px;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  padding: 0 3px;
  height: 16px;
  min-width: 16px;
  border-radius: 16px;
  font: var(--fontSizeCaption);
  background: var(--actionColor);
  color: var(--badgeForegroundColor);
}
</style>
