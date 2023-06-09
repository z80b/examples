<template>
  <x-link :class="$style.root" :href="categoryData.link || '/'">
    <div :class="$style.container">
      <div :class="$style.background" :style="{ backgroundImage }"></div>
      <span :class="[$style.title, { [$style.titleWhite]: isTitleWhite }]">
        {{ categoryData.title }}
      </span>
    </div>
  </x-link>
</template>

<script setup lang="ts">
import { computed } from 'vue';

import type { PropType } from 'vue';
import type { WidgetMenuItem } from 'api-gen/models/widget-menu-item';

import XLink from 'blocks/x-link/x-link.vue';

const props = defineProps({
  categoryData: {
    type: Object as PropType<WidgetMenuItem>,
    required: true,
  },
});

const backgroundImage = computed(() => `url(${props.categoryData.image})`);

// FIXME title_white - актуальный ключ?
// @ts-ignore
const isTitleWhite = computed(() => props.categoryData.titleWhite || props.categoryData.title_white);
</script>

<style module>
.root {
  display: inline-block;
  width: 172px;
  height: 68px;
  overflow: hidden;
  border-radius: 4px;
  transition: box-shadow var(--animationDuration);
}

.container {
  display: flex;
  align-items: center;
  width: 100%;
  height: 100%;
  position: relative;
}

.container::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  transform: translateX(-100%);
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
  transition: transform 0.8s, box-shadow 0.8s;
  transition-delay: 0.5s;
  opacity: 0;
}

.root:hover .container::after {
  transform: translateX(100%);
  opacity: 1;
}

.background {
  background-repeat: no-repeat;
  position: absolute;
  background-position: 50% 0;
  background-size: cover;
  height: 100%;
  width: 100%;
  transition: transform var(--animationDuration) ease-out;
}

.title {
  font: var(--fontSizeSubLine);
  margin-left: 24px;
  color: var(--labelColor);
  z-index: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: calc(100% - 60px);
}

.titleWhite {
  color: var(--badgeForegroundColor);
}

.root:hover .background {
  transform: scale(1.2);
}

@media (--wide-viewport) {
  .root {
    width: 220px;
  }

  .background {
    background-position: 0 0;
  }
}
</style>
