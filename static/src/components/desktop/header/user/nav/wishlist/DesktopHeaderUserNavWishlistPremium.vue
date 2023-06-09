<template>
  <x-link
    :class="$style.root"
    themes="secondaryLabel"
    href="/wishlist/"
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <x-icon
      :type="iconType"
      :class="theme && $style[`icon_${theme}`]"
      :size="24"
    />
    <span
      :class="[
        $style.text,
        theme === 'alternative' ? $style.textSecondary : $style.textDefault,
      ]">
      {{ $t('favorites') }}
    </span>
  </x-link>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

import iconTypes from 'blocks/x-icon/icon-types';
import XIcon from 'blocks/x-icon/x-icon.vue';
import XLink from 'blocks/x-link/x-link.vue';

import type { PropType } from 'vue';

const props = defineProps({
  theme: {
    type: String as PropType<'alternative' | ''>,
    default: '',
  },
  isActive: {
    type: Boolean,
    default: false,
  },
});

const isHovered = ref(false);

const iconType = computed(() => {
  if (isHovered.value || props.isActive) {
    return iconTypes.wishlistPremiumHover;
  }
  return iconTypes.wishlistPremium;
});
</script>

<style module>
  .root {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .icon_alternative {
    filter: invert(1);
  }

  .text {
    font: var(--fontSizeCaption);
    margin-top: 4px;
  }

  .textDefault {
    color: var(--secondaryLabelColor);
  }

  .textSecondary {
    color: var(--overlayLightColor50);
  }

  .root:hover .textDefault {
    color: var(--labelColor);
  }

  .root:hover .textSecondary {
    color: var(--badgeForegroundColor);
  }
</style>
