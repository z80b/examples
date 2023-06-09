<template>
  <div v-if="categoriesData" :class="$style.root">
    <x-scrolling-content-on-hover :key="categoriesData.length" :options="{ spaceBetween: 24 }">
      <SwiperSlide
        v-for="(item, key) in categoriesData"
        :key="key"
        :class="$style.swiperSlide"
      >
        <DesktopHeaderPremiumTopmenuCategoriesItem :category-data="item" />
      </SwiperSlide>
    </x-scrolling-content-on-hover>
  </div>
</template>

<script setup lang="ts">
import type { PropType } from 'vue';
import type { WidgetMenuItem } from 'api-gen/models/widget-menu-item';

import { SwiperSlide } from 'swiper/vue';
import XScrollingContentOnHover from 'blocks/x-scrolling-content-on-hover/x-scrolling-content-on-hover.vue';
import DesktopHeaderPremiumTopmenuCategoriesItem from 'components/desktop/header/topmenu/DesktopHeaderPremiumTopmenuCategoriesItem.vue';

defineProps({
  categoriesData: {
    type: Array as PropType<WidgetMenuItem[] | null>,
    required: true,
  },
});
</script>

<style module>
.root {
  font-size: 0;
  position: relative;
}

.root::after {
  content: '';
  position: absolute;
  top: -15px;
  left: -20px;
  background: var(--headerPremiumCategoriesBackground);
  width: calc(100% + 40px);
  height: 100%;
}

.root .swiperSlide {
  width: auto;
  padding-top: 8px;
}

.swiperSlide:first-child {
  margin-left: 0;
}

@media (--desktop-viewport) {
  /* Растягиваем бекграунд для сервисов на весь экран внутри контейнера и центрируем */
  .root::after {
    left: calc(-50vw + var(--desktopLayout) / 2);
    width: 100vw;
  }
}

@media (--wide-viewport) {
  .root::after {
    left: calc(-50vw + var(--maxLayout) / 2);
  }
}
</style>
