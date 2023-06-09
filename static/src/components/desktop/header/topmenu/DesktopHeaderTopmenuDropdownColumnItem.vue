<template>
  <x-link
    :to="itemUrl"
    :class="classList"
    :themes="cmsClass === cmsItemClasses.more ? 'link' : 'label'"
    :disabled="!itemUrl"
    @click="onClick(itemUrl)"
  >
    <slot></slot>
  </x-link>
</template>

<script setup lang="ts">
import { computed, useCssModule } from 'vue';
import { useAppStore } from 'stores/app';
import { useStats } from 'composables/useStats';
import { useRoute } from 'vue-router';
import { TopmenuCategoryClass, cmsItemClasses } from 'blocks/common/topmenu';
import { EVENTS } from 'types/events';
import { addQueryToUrl } from 'blocks/common/utils/query';
import { useTopMenuStats } from 'composables/statistics/useTopMenuStats';

import XLink from 'blocks/x-link/x-link.vue';

const props = defineProps({
  cmsClass: {
    type: String,
    default: '',
  },
  isPremium: {
    type: Boolean,
    default: false,
  },
  meta: {
    type: Object,
    default: () => ({}),
  },
  href: {
    type: String,
    default: '',
  },
  level: {
    type: Number,
    required: true,
  },
  title: {
    type: String,
    default: '',
  },
  categoryTitle: {
    type: String,
    default: '',
  },
});

const style = useCssModule();
const route = useRoute();
const stats = useStats();

const appStore = useAppStore();

const { reportTopMenuLinkClick } = useTopMenuStats();

const classList = computed(() => ({
  [style.link]: true,
  [style.premium]: props.isPremium,
  [style.linkBold]: props.cmsClass === cmsItemClasses.bold,
  [style.linkTitle]: props.cmsClass === cmsItemClasses.title,
  [style.linkMore]: props.cmsClass === cmsItemClasses.more,
}));

const itemUrl = computed(() => (
  props.href
    ? addQueryToUrl(props.href, {
      sitelink: `topmenu${appStore.shortGender.toUpperCase()}`,
      l: props.level + 1,
    })
    : null));

const onClick = (url?: string | null) => {
  reportTopMenuLinkClick({
    element: props.title,
    parentCategory: props.categoryTitle,
    gender: appStore.gender,
    url,
    previousUrl: route.fullPath,
  });

  if (props.cmsClass === TopmenuCategoryClass.Just4You) {
    stats.report(EVENTS.JUSTFORYOU_OPEN_COLLECTION, {
      source: 'menu_page',
      uuid: props.meta?.uuid,
    });
  }
};

</script>

<style module>
.root {
  display: inline-block;
}

.link:hover {
  color: var(--secondaryLabelColor);
}

.link:active {
  color: var(--labelColor);
}

.linkBold {
  position: relative;
}

.linkBold::before {
  content: '';
  position: absolute;
  top: 5px;
  left: -8px;
  width: 4px;
  height: 4px;
  background-color: var(--labelColor);
  border-radius: 50%;
}

.premium.linkBold::before {
  top: 7px;
}

.linkTitle {
  font-weight: bold;
  line-height: 18px;
}
</style>
