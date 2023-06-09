<!-- Компонент, контролирующий видимость строки поиска на Backbone в хэдере -->

<template>
  <div :class="$style.root">
    <DesktopHeaderSearchPremium
      v-if="$experiments.test('ITPMO918_WEB_Header') || appStore.isTransparentHeader"
      :stretch="stretch"
      @update:search="$emit('update:search', $event)"
    />
    <DesktopHeaderSearchDefaultWeb6176
      v-else-if="$experiments.test('WEB6176_search_bar_2')"
      @update:search="$emit('update:search', $event)"
    />
    <DesktopHeaderSearchDefault
      v-else
      @update:search="$emit('update:search', $event)"
    />
  </div>
</template>

<script setup lang="ts">
import { defineAsyncComponent } from 'vue';
import { useAppStore } from 'stores/app';

import DesktopHeaderSearchDefault from 'components/desktop/header/search/DesktopHeaderSearchDefault.vue';
import DesktopHeaderSearchPremium from 'components/desktop/header/search/DesktopHeaderSearchPremium.vue';

const DesktopHeaderSearchDefaultWeb6176 = defineAsyncComponent(() => import('components/desktop/header/search/DesktopHeaderSearchDefaultWeb6176.vue'));

defineEmits(['update:search']);

defineProps({
  stretch: {
    type: Boolean,
    default: false,
  },
});

const appStore = useAppStore();
</script>

<style module>
  .root {
    margin-left: 32px;
    flex-grow: 1;
    position: relative;
    z-index: 1;
  }
</style>
