<template>
  <div
    :class="[
      isOldDskSearch
        ? [
          $style.oldListItem,
          {[$style.oldListItemSelected]: selectedIndex === index},
        ]
        : [
          $style.listItem,
          {[$style.listItemSelected]: selectedIndex === index},
        ],
      isSmallerPaddings && $style.listItemSmaller
    ]"
    @mouseenter="onMouseEnter"
    @mousedown="$emit('mousedown', index)"
  >
    <span :class="isOldDskSearch ? $style.oldListItemText : $style.listItemText">{{ text }}
      <x-badges
        v-if="isPremium"
        :badges="[{ type: 'premium', text: 'premium' }]"
        :class="$style.recentBadge"
      />
    </span>
    <div
      v-if="!isOldDskSearch"
      :class="$style.listItemRemove"
      @mousedown="$emit('remove', index)"
    >
      <x-icon
        :type="isResetHovered ? XIcon.types.crossBlack : XIcon.types.crossThinBlack"
        :size="16"
        @mouseenter="(event: Event) => onMouseEnter(event, true)"
        @mouseleave="isResetHovered = false"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

import XIcon from 'blocks/x-icon/x-icon.vue';
import XBadges from 'blocks/x-badges/x-badges.vue';

defineProps({
  index: {
    type: Number,
    default: 0,
  },
  selectedIndex: {
    type: Number,
    default: 0,
  },
  text: {
    type: String,
    default: '',
  },
  isOldDskSearch: {
    type: Boolean,
    default: false,
  },
  isSmallerPaddings: {
    type: Boolean,
    default: false,
  },
  isPremium: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(['mousedown', 'remove', 'mouseenter']);

const isResetHovered = ref(false);

const onMouseEnter = (event: Event, isHovered: boolean) => {
  if (isHovered) {
    isResetHovered.value = true;
  }

  emit('mouseenter', event);
};
</script>

<style module>
  .oldListItem {
    font: var(--fontSize);
    background-color: var(--backgroundColor);
    cursor: pointer;
    transition: all 0.2s linear;
    padding: 8px 0;
  }

  .oldListItemSelected,
  .oldListItem:focus {
    background-color: var(--secondaryColor);
  }

  .oldListItemText {
    display: inline-block;
    flex-grow: 1;
    color: var(--labelColor);
    text-decoration: none;
    word-break: break-word;
    padding: 0 20px;
  }

  .listItem {
    display: flex;
    align-items: center;
    justify-content: space-between;
    cursor: pointer;
  }

  .listItemSelected {
    background-color: var(--secondaryBackgroundColor);
  }

  .listItem:active,
  .listItem:focus {
    background-color: var(--secondaryBackgroundColor);
  }

  .listItemText {
    flex-grow: 1;
    color: var(--labelColor);
    text-decoration: none;
    word-break: break-word;
    padding: 10px 0 10px 16px;
  }

  .listItemRemove {
    display: inline-flex;
    padding: 12px 0 12px 12px;
    margin-right: 4px;
  }

  .recentBadge {
    margin-left: 8px;
  }

  @media (--desktop-viewport) {
    .listItem {
      padding-left: 16px;
      padding-right: 16px;
    }

    .listItemText {
      padding-left: 0;
    }

    .listItemSmaller {
      padding: 8px 10px;
    }

    .listItemSmaller .listItemText {
      padding: 0;
    }

    .listItemSmaller .listItemRemove {
      padding: 0;
      margin: 0;
    }
  }
</style>
