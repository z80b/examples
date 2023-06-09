<template>
  <x-link
    :to="categoryUrl"
    :class="classList"
    :disabled="!categoryUrl"
    themes="custom"
  >
    <x-discovery-menu-animation
      v-if="category.class === TopmenuCategoryClass.Discovery"
      :class="$style.animation"
    />
    {{ category.title?.text }}
  </x-link>
</template>

<script setup lang="ts">
import { computed, useCssModule } from 'vue';
import { useAppStore } from 'stores/app';
import { addQueryToUrl } from 'blocks/common/utils/query';
import { TopmenuCategoryClass } from 'blocks/common/topmenu';

import type { PropType } from 'vue';
import type { TopmenuCategory } from 'types/topmenu';

import XLink from 'blocks/x-link/x-link.vue';
import XDiscoveryMenuAnimation from 'blocks/x-discovery/x-discovery-menu-animation.vue';

const props = defineProps({
  category: {
    type: Object as PropType<TopmenuCategory>,
    required: true,
  },
  level: {
    type: Number,
    required: true,
  },
  itemHovered: {
    type: String,
    default: '',
  },
  isMenuHovered: {
    type: Boolean,
    default: false,
  },
  isPremium: {
    type: Boolean,
    default: false,
  },
});

const appStore = useAppStore();
const style = useCssModule();

const classList = computed(() => ({
  [style.link]: !props.isMenuHovered,
  [style.linkPremium]: props.isPremium && (props.category.class !== TopmenuCategoryClass.Red),
  [style.linkHovered]: props.isMenuHovered,
  [style.linkRed]: props.category.class === TopmenuCategoryClass.Red,
  [style.linkActive]: props.category.title?.text === props.itemHovered && !(props.category.class === TopmenuCategoryClass.Red),
}));

const categoryUrl = computed(() => (props.category.title?.url
  ? addQueryToUrl(props.category.title?.url, {
    sitelink: `topmenu${appStore.shortGender.toUpperCase()}`,
    l: props.level + 1,
  })
  : null));
</script>

<style module>
.link {
  display: inline-block;
  position: relative;
  color: var(--labelColor);
}

.link:hover {
  color: var(--secondaryLabelColor);
}

.linkHovered,
.linkHovered:visited {
  display: inline-block;
  position: relative;
  color: var(--secondaryLabelColor);
}

.linkHovered:hover {
  color: var(--labelColor);
}

.linkHovered:active {
  color: var(--labelColor);
}

.linkRed,
.linkRed:hover,
.linkRed:visited {
  color: var(--actionColor);
}

.linkActive,
.linkActive:hover,
.linkActive:visited {
  color: var(--labelColor);
}

.linkPremium {
  color: var(--backgroundColor);
}

.linkPremium:hover,
.linkPremium:visited {
  color: var(--overlayLightColor50);
}

.animation {
  position: absolute;
  top: 50%;
  margin-top: -4px;
  margin-left: -12px;
}
</style>
