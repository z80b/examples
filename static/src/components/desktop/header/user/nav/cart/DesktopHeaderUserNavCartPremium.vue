<template>
  <x-link
    :class="$style.root"
    themes="secondaryLabel"
    href="/checkout/cart/"
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
      {{ $t('cart') }}
    </span>
    <span v-if="cartTotalQuantity" :class="$style.count">
      {{ cartTotalQuantity }}
    </span>
  </x-link>
</template>

<script setup lang="ts">
import { PropType, ref, computed } from 'vue';
import { useStore } from 'vuex';
import * as GETTERS from 'blocks/common/store/getter-types';

import iconTypes from 'blocks/x-icon/icon-types';
import XIcon from 'blocks/x-icon/x-icon.vue';
import XLink from 'blocks/x-link/x-link.vue';

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

const store = useStore();

const isHovered = ref(false);

const cartTotalQuantity = computed(() => store.getters[GETTERS.CART_TOTAL_QUANTITY]);

const iconType = computed(() => {
  if (isHovered.value || props.isActive) {
    return iconTypes.cartPremiumHover;
  }
  return iconTypes.cartPremium;
});
</script>

<style module>
  .root {
    cursor: pointer;
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

  .count {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    position: absolute;
    top: -7px;
    right: 7px;
    color: var(--badgeForegroundColor);
    background-color: var(--actionColor);
    font: var(--fontSizeCaption);
    text-align: center;
  }
</style>
