<template>
  <x-link :class="$style.root" :to="category?.title?.url">
    <div :class="[$style.button, isOnPremiumPage && $style.buttonSecondary]">
      <div :class="$style.content">
        <span :class="$style.title">Premium</span>
        <div :class="$style.arrow" :style="style"></div>
      </div>
      <div :class="$style.subtitle">{{ $t('premium-topmenu-button-subtitle') }}</div>
    </div>
  </x-link>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { imageUrlCms } from 'blocks/common/utils/formatters';
import { useSite } from 'composables/useSite';

import type { PropType } from 'vue';
import type { TopmenuCategory } from 'types/topmenu';

import XLink from 'blocks/x-link/x-link.vue';

defineProps({
  category: {
    type: Object as PropType<TopmenuCategory>,
    required: true,
  },
  isOnPremiumPage: {
    type: Boolean,
    default: false,
  },
});

const site = useSite();

const style = computed(() => ({
  backgroundImage: `url(${imageUrlCms('header-premium-arrow-long.svg', site.mediaCdn)})`,
}));
</script>

<style module>
.root {
  display: block;
  margin-right: 52px;
}

.button {
  width: 160px;
  height: 100%;
  background-color: var(--secondaryBackgroundColor);
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
}

.content {
  display: flex;
  font: var(--fontSizeSubLine);
  color: var(--labelColor);
  padding-top: 8px;
  padding-left: 16px;
  background-color: inherit;
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
}

.title {
  padding-right: 4px;
  background-color: inherit;
  z-index: 1;
}

.subtitle {
  font: var(--fontSizeCaption);
  color: var(--secondaryLabelColor);
  padding: 0 16px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.arrow {
  display: inline-block;
  width: 46px;
  height: 20px;
  transition: transform var(--animationDuration) ease;
  background-position: 0 2px;
  background-repeat: no-repeat;
  transform: translateX(-30px);
}

.buttonSecondary .arrow {
  filter: invert(1);
}

.button:hover .arrow {
  transform: translateX(0);
}

.buttonSecondary {
  background-color: var(--labelColor);
}

.buttonSecondary .content {
  color: var(--badgeForegroundColor);
}

@media (--wide-viewport) {
  .root {
    margin-right: 82px;
  }
}
</style>
