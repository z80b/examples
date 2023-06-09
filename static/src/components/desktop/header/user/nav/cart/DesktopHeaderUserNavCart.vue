<template>
  <x-link
    :class="$style.root"
    themes="secondaryLabel"
    href="/checkout/cart/"
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <x-icon
      :class="$style.icon"
      :type="isHovered ? iconTypes.headerNavCartActive : iconTypes.headerNavCart"
      :size="24"
    />
    <span v-if="cartTotalQuantity" :class="$style.count">
      {{ cartTotalQuantity }} {{ $t('products__pluralize', cartTotalQuantity) }}
    </span>
    <span v-else :class="$style.count">{{ $t('cart') }}</span>
  </x-link>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useStore } from 'vuex';
import * as GETTERS from 'blocks/common/store/getter-types';

import XIcon from 'blocks/x-icon/x-icon.vue';
import iconTypes from 'blocks/x-icon/icon-types';
import XLink from 'blocks/x-link/x-link.vue';

const store = useStore();

const isHovered = ref(false);

const cartTotalQuantity = computed(() => store.getters[GETTERS.CART_TOTAL_QUANTITY]);
</script>

<style module>
  .root {
    display: flex;
    align-items: flex-end;
  }

  .icon {
    margin-right: 4px;
  }

  .count {
    color: var(--labelColor);
  }
</style>
