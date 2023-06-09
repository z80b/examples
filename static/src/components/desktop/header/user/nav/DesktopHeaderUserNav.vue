<template>
  <div :class="$style.root">
    <template v-if="isLogon">
      <DesktopHeaderUserNavProfile :class="$style.link">
        <x-link
          href="/sales/order/history/"
          themes="secondaryLabel"
          @click="addNoPhoneCookie"
        >
          {{ $t('profile') }}
        </x-link>
      </DesktopHeaderUserNavProfile>
      <x-link
        :class="$style.link"
        themes="secondaryLabel"
        href="/wishlist/"
      >
        {{ $t('favorites') }}
      </x-link>
    </template>
    <template v-else>
      <x-link
        :class="$style.link"
        themes="secondaryLabel"
        @click="showLoginModal({ place: 'topbar' })"
      >
        {{ $t('login') }}
      </x-link>
    </template>
    <DesktopHeaderUserNavCart :class="$style.link" />
  </div>
</template>

<script setup lang="ts">
import { useStorage } from 'composables/useStorage';
import { useAuth } from 'composables/useAuth';
import { CookieStorageKey } from 'types/storage';

import DesktopHeaderUserNavProfile from 'components/desktop/header/user/nav/profile/DesktopHeaderUserNavProfile.vue';
import DesktopHeaderUserNavCart from 'components/desktop/header/user/nav/cart/DesktopHeaderUserNavCart.vue';
import XLink from 'blocks/x-link/x-link.vue';

const storage = useStorage();
const { isLogon, showLoginModal } = useAuth();

const addNoPhoneCookie = () => {
  storage.setCookie(CookieStorageKey.NO_PHONE, true);
};
</script>

<style module>
  .root {
    display: flex;
    flex-flow: row nowrap;
    align-items: flex-end;
  }

  .link {
    margin-left: 12px;
    font: var(--fontSizeSubLine);
  }
</style>
