<template>
  <x-popover
    :class="$style.root"
    :triggers="[]"
    :shown="isOpen"
    position="bottom"
    :flip="false"
    :distance="$experiments.get('web_8401_lp_redesign') === 'show' ? 12 : undefined"
    :theme="$experiments.get('web_8401_lp_redesign') === 'show' ? 'popover-no-arrow' : 'popover'"
    eager-mount
    @mouseenter-content="onMouseEnter"
    @mouseleave-content="onMouseLeave"
    @mouseenter-target="onMouseEnter"
    @mouseleave-target="onMouseLeave"
    @show="onPopoverShow"
  >
    <span role="link" :class="$style.brokenUnhoverFix">
      <x-link
        :class="$style.button"
        themes="secondaryLabel"
        href="/sales/order/history/"
        @click="addNoPhoneCookie"
      >
        <x-icon
          :type="iconType"
          :class="theme && $style[`icon_${theme}`]"
          :size="24"
        />
        <span v-if="loyaltyStore.actions" :class="$style.discount">
          %
        </span>
        <span
          :class="[
            $style.text,
            $style[props.theme],
            isOpen && $style[`${theme}_hover`],
          ]">
          {{ $t('profile') }}
        </span>
      </x-link>
    </span>
    <template #content>
      <DesktopHeaderUserNavProfileContent @close="close" />
    </template>
  </x-popover>
</template>

<script setup lang="ts">
import { defineAsyncComponent, ref, computed } from 'vue';
import { promiseTimeout } from '@vueuse/core';
import { useLoyaltyStore } from 'stores/loyalty';
import { useStorage } from 'composables/useStorage';
import { CookieStorageKey } from 'types/storage';
import { getLeaveElementSide } from 'blocks/common/utils/dom';

import XIcon from 'blocks/x-icon/x-icon.vue';
import XLink from 'blocks/x-link/x-link.vue';
import XPopover from 'blocks/x-popover/x-popover.vue';

import type { PropType } from 'vue';

// NOTE: Компонент сделан асинхронным, чтобы обойти баг в сборке (аналог WEB-10012)
const DesktopHeaderUserNavProfileContent = defineAsyncComponent(() => import('components/desktop/header/user/nav/profile/DesktopHeaderUserNavProfileContent.vue'));

const storage = useStorage();

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

const loyaltyStore = useLoyaltyStore();

const isOpen = ref(false);
const isPopoverShown = ref(false);

const iconType = computed(() => {
  const iconTypeName = (isOpen.value || props.isActive) ? 'profilePremiumHover' : 'profilePremium';
  return XIcon.types[iconTypeName];
});

const close = () => {
  isOpen.value = false;
};

const onMouseEnter = () => {
  isOpen.value = true;
};

const onMouseLeave = async (event: MouseEvent) => {
  if (getLeaveElementSide(event) !== 'bottom') {
    await promiseTimeout(300);
    close();
  }
};

const addNoPhoneCookie = () => {
  storage.setCookie(CookieStorageKey.NO_PHONE, true);
};

const onPopoverShow = () => {
  if (!isPopoverShown.value) {
    isPopoverShown.value = true;
  }
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

.button {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.discount {
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

/* THEMES */
.icon_alternative {
  filter: invert(1);
}

.text {
  font: var(--fontSizeCaption);
  margin-top: 4px;
}

.default {
  color: var(--secondaryLabelColor);
}

.default_hover {
  color: var(--labelColor);
}

.alternative {
  color: var(--overlayLightColor50);
}

.alternative_hover {
  color: var(--badgeForegroundColor);
}

@media (--desktop-hover) {
  .item:hover {
    color: var(--labelColor);
  }
}
</style>
