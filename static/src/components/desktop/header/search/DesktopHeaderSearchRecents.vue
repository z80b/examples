<template>
  <div
    v-if="recentsToShow.length"
    :class="$style.root"
  >
    <div :class="isOldDskSearch ? $style.oldHeader : $style.header">
      <span :class="isOldDskSearch ? $style.oldTitle : $style.title">{{ $t('previously_you_searched') }}</span>
      <x-link
        v-if="$device.isMobileOrTablet"
        themes="link"
        @mousedown="clear"
      >{{ $t('clear') }}</x-link>
    </div>
    <div :class="$style.list">
      <DesktopHeaderSearchRecent
        v-for="(recent, index) in recentsToShow"
        :key="recent.text"
        :is-old-dsk-search="isOldDskSearch"
        :index="index"
        :text="recent.text"
        :selected-index="selectedIndex"
        @mouseenter="(event) => $emit('mouseenter', index, event)"
        @mousedown="$emit('mousedown', index)"
        @remove="remove(recent, index)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { EVENTS } from 'types/events';
import { useSearchRecents } from 'composables/useSearchRecents';
import { useDevice } from 'composables/useDevice';
import { useStats } from 'composables/useStats';

import XLink from 'blocks/x-link/x-link.vue';
import DesktopHeaderSearchRecent from 'components/desktop/header/search/DesktopHeaderSearchRecent.vue';

const device = useDevice();
const stats = useStats();

defineProps({
  selectedIndex: {
    type: Number,
    default: -1,
  },
  isOldDskSearch: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(['mouseenter', 'mousedown', 'action']);

const { recents, recentsToShow, removeRecentEntry, clearRecentEntries } = useSearchRecents({ showLimit: device.isWideOrDesktop ? 10 : 5 });

const clear = () => {
  clearRecentEntries();
  emit('action');
};

const remove = ({ text }: { text: string}, index: number) => {
  emit('action');
  removeRecentEntry(text);

  stats.report(EVENTS.SEARCH_REMOVE_RECENT, {
    elementType: 'search_bar',
    query: text,
    suggest: {
      list: recents.value,
      type: 'recent_suggest',
    },
    newSuggest: {
      list: recents.value,
    },
    item: `${index}:${text}`,
  });
};
</script>

<style module>
  .oldHeader {
    padding: 8px 20px;
    background-color: var(--backgroundColor);
    cursor: pointer;
    transition: all 0.2s linear;
  }

  .oldTitle {
    font: var(--fontSize);
    color: var(--secondaryLabelColor);
    cursor: default;
  }

  .root {
    padding: 18px 0 16px 0;
  }

  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding-left: 16px;
    padding-right: 16px;
    width: 100%;
    box-sizing: border-box;
  }

  .title {
    font: var(--fontSizeSubLine);
    font-weight: bold;
    cursor: default;
  }

  .list {
    /* вычитаем нижний padding от root для правильного определения высоты */
    height: calc(100% - 16px);
    width: 100%;
    font: var(--fontSizeSubLine);
    display: flex;
    flex-direction: column;
    background-color: var(--badgeForegroundColor);
    overflow-y: auto;
    overflow-x: hidden;
  }

  @media (--desktop-viewport) {
    .root {
      padding: 0;
    }

    .header {
      margin-bottom: 0;
      padding: 10px 0 10px 16px;
    }

    .title {
      font-weight: normal;
      color: var(--secondaryLabelColor);
    }
  }
</style>
