<template>
  <div :class="[$style.root, $style.col]">
    <DesktopHeaderTopmenuDropdownColumnItem
      v-if="titleItem"
      :id="titleItemId"
      :cms-class="titleItem.class"
      :is-premium="isPremium"
      :class="[$style.link, isPremium && $style.title_premium]"
      role="heading"
      :meta="meta"
      :href="titleItem.url"
      :title="titleItem.text"
      :category-title="categoryTitle"
      :level="0"
    >
      {{ titleItem.text }}
    </DesktopHeaderTopmenuDropdownColumnItem>
    <div
      v-else-if="allowEmptyTitle"
      :class="[
        $style.link,
        $style.titlePlaceholder,
        isPremium && $style.titlePlaceholder_premium
      ]">
    </div>
    <div
      v-if="titleItem"
      role="region"
      :aria-labelledby="titleItemId"
      :class="$style.col"
    >
      <DesktopHeaderTopmenuDropdownColumnItem
        v-for="(linkItem, level) in linkItems"
        :key="`submenu_${level}_${linkItem.text}`"
        :level="level + 1"
        :href="linkItem.url"
        :title="linkItem.text"
        :category-title="categoryTitle"
        :cms-class="linkItem.class"
        :is-premium="isPremium"
        :class="[$style.link, isPremium && $style.link_premium]"
        :meta="meta"
      >
        {{ linkItem.text }}
      </DesktopHeaderTopmenuDropdownColumnItem>
    </div>
    <template v-else>
      <DesktopHeaderTopmenuDropdownColumnItem
        v-for="(linkItem, level) in linkItems"
        :key="`submenu_${level}_${linkItem.text}`"
        :cms-class="linkItem.class"
        :is-premium="isPremium"
        :level="level"
        :href="linkItem.url"
        :title="linkItem.text"
        :category-title="categoryTitle"
        :class="[$style.link, isPremium && $style.link_premium]"
        :meta="meta"
      >
        {{ linkItem.text }}
      </DesktopHeaderTopmenuDropdownColumnItem>
    </template>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { cmsItemClasses } from 'blocks/common/topmenu';
import uniqueId from 'lodash/uniqueId';

import type { PropType } from 'vue';
import type { TopmenuItem } from 'types/topmenu';

import DesktopHeaderTopmenuDropdownColumnItem from 'components/desktop/header/topmenu/DesktopHeaderTopmenuDropdownColumnItem.vue';

const props = defineProps({
  categoryTitle: {
    type: String,
    default: '',
  },
  items: {
    type: Array as PropType<TopmenuItem[]>,
    required: true,
  },
  allowEmptyTitle: {
    type: Boolean,
    default: true,
  },
  isPremium: {
    type: Boolean,
    default: false,
  },
  meta: {
    type: Object,
    default: () => ({}),
  },
});

const linkItems = computed(() => props.items.filter(item => item.class !== cmsItemClasses.title && item.text));
const titleItem = computed(() => props.items.find(item => item.class === cmsItemClasses.title && item.text));
const titleItemId = computed(() => (titleItem.value ? uniqueId(titleItem.value.text) : ''));
</script>

<style module>
  .root {
    line-height: 18px;
    font-size: 13px;
    width: 198px;
    margin: 0 8px;
  }

  .col {
    display: flex;
    flex-flow: column nowrap;
    align-items: flex-start;
    justify-content: flex-start;
  }

  .titlePlaceholder {
    height: 18px;
  }

  .link {
    padding-bottom: 8px;
  }

  .title_premium {
    font: var(--fontSizeHeadLine);
    line-height: 28px;
    padding-bottom: 16px;
  }

  .link_premium {
    font: var(--fontSize);
  }

  .titlePlaceholder_premium {
    height: 28px;
    padding-bottom: 20px;
  }

  @media (--wide-viewport) {
    .root {
      width: 280px;
    }

    .title_premium {
      font: var(--fontSizeHeadLine);
      padding-bottom: 16px;
    }

    .link_premium {
      font: var(--fontSizeSubLine);
    }
  }
</style>
