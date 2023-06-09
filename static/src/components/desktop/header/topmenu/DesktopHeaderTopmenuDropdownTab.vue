<template>
  <x-transition-fade
    mode="out-in"
    speed="fast"
    appear
  >
    <div :class="$style.root">
      <template v-for="column, index in columns" :key="index">
        <DesktopHeaderTopmenuDropdownColumn
          v-if="column.items"
          :class="$style.col"
          :category-title="title"
          :items="filterDisabled(column.items)"
          :allow-empty-title="allowEmptyTitle"
          :meta="meta"
        />
        <x-transition-fade
          v-else-if="column.displayType === 'banner'"
          mode="out-in"
          speed="fast"
        >
          <x-banner
            :class="{
              [$style.banner]: true,
              [$style.bannerFull]: !columns.length
            }"
            :type="column.banner!.bannerSlot"
            @click="onBannerClick(column.banner!.bannerSlot)"
          />
        </x-transition-fade>
      </template>
    </div>
  </x-transition-fade>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useAppStore } from 'stores/app';
import { useRoute } from 'vue-router';
import { useTopMenuStats } from 'composables/statistics/useTopMenuStats';

import type { PropType } from 'vue';
import type { DisabledClasses, TopmenuTab, TopmenuItem } from 'types/topmenu';

import XBanner from 'blocks/x-banner/x-banner.vue';
import XTransitionFade from 'blocks/x-transition-fade/x-transition-fade.vue';
import DesktopHeaderTopmenuDropdownColumn from 'components/desktop/header/topmenu/DesktopHeaderTopmenuDropdownColumn.vue';

const props = defineProps({
  tab: {
    type: Object as PropType<TopmenuTab>,
    required: true,
  },
  title: {
    type: String,
    default: '',
  },
  allowEmptyTitle: {
    type: Boolean,
    default: true,
  },
  disabledClasses: {
    type: Array as PropType<DisabledClasses>,
    default: () => [],
  },
  meta: {
    type: Object,
    default: () => ({}),
  },
});

const route = useRoute();

const appStore = useAppStore();

const { reportTopMenuLinkClick } = useTopMenuStats();

const columns = computed(() => props.tab?.top?.groups?.data?.flat() ?? []);

const filterDisabled = (items: TopmenuItem[]) => items.filter(item => !props.disabledClasses.includes(item.class as any));

const onBannerClick = (banner: string) => {
  reportTopMenuLinkClick({
    element: banner || '',
    parentCategory: props.title,
    isBanner: true,
    gender: appStore.gender,
    previousUrl: route.fullPath,
  });
};
</script>

<style module>
  .root {
    padding-top: 10px;
    display: flex;
    flex-flow: row nowrap;
    justify-content: flex-start;
    flex-grow: 1;
  }

  .col {
    margin: 0 15px;
  }

  .col:first-child {
    margin-left: 0;
  }

  .banner {
    justify-content: flex-end;
    flex-grow: 1;
    display: flex;
    white-space: nowrap;
  }

  .bannerFull {
    display: block;
  }
</style>
