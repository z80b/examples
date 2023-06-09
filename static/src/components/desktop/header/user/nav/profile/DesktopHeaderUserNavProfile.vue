<template>
  <x-popover
    :class="$style.root"
    :triggers="[]"
    :shown="isOpen"
    position="bottom-start"
    strategy="fixed"
    :flip="false"
    eager-mount
    :distance="$experiments.get('web_8401_lp_redesign') === 'show' ? 12 : undefined"
    :theme="$experiments.get('web_8401_lp_redesign') === 'show' ? 'popover-no-arrow' : 'popover'"
    @mouseleave-content="isOpen = false"
    @mouseleave-target="onLinkLeave"
    @show="onPopoverShow"
  >
    <span
      role="link"
      :class="$style.brokenUnhoverFix"
      @mouseover="isOpen = true"
    >
      <slot></slot>
      <x-badges :class="$style.discountBadges" :badges="discountBadges" />
    </span>
    <template #content>
      <DesktopHeaderUserNavProfileContent @close="close" />
    </template>
  </x-popover>
</template>

<script setup lang="ts">
import { defineAsyncComponent, ref, computed } from 'vue';
import { useLoyaltyStore } from 'stores/loyalty';
import { getLeaveElementSide } from 'blocks/common/utils/dom';

import XPopover from 'blocks/x-popover/x-popover.vue';
import XBadges from 'blocks/x-badges/x-badges.vue';

// NOTE: Компонент сделан асинхронным, чтобы обойти баг в сборке (аналог WEB-10012)
const DesktopHeaderUserNavProfileContent = defineAsyncComponent(() => import('components/desktop/header/user/nav/profile/DesktopHeaderUserNavProfileContent.vue'));

const loyaltyStore = useLoyaltyStore();

const isOpen = ref(false);
const isPopoverInit = ref(false);

const discountBadges = computed(() => {
  const arr = [];

  if (loyaltyStore.discount > 0) {
    arr.push({
      format: 'discount',
      type: 'discount',
      text: loyaltyStore.discount,
    });
  }

  const [firstAction] = loyaltyStore.actions;
  const additionalDiscount = firstAction?.discounts?.[0].discount ?? 0;
  if (additionalDiscount > 0) {
    arr.push({
      format: 'discount',
      type: 'discount',
      text: additionalDiscount,
    });
  }

  return arr;
});

const close = () => {
  isOpen.value = false;
};

const onPopoverShow = () => {
  if (!isPopoverInit.value) {
    isPopoverInit.value = true;
  }
};

const onLinkLeave = (event: MouseEvent) => {
  if (getLeaveElementSide(event) !== 'bottom') close();
};
</script>

<style module>
  .root {
    display: inline-block;
    position: relative;
  }

  /* Фиксим незакрытие поповера при переходе между профилем и контентом, фиктивно расширяя блок */
  .brokenUnhoverFix {
    padding-bottom: 12px;
    margin-bottom: -12px;
  }

  .discountBadges {
    position: relative;
    top: -12px;
    left: -4px;
  }
</style>
