<template>
  <div :class="$style.root">
    <DesktopHeaderUserNavProfilePremium
      v-if="isLogon"
      :class="$style.profile"
      :theme="userNavItemsTheme"
      :is-active="isCustomerPage"
    />
    <x-button
      v-else
      :class="$style.button"
      :theme="buttonTheme"
      :intrinsic-width="false"
      @click="showLoginModal({ place: 'topbar' })"
    >
      {{ $t('login') }}
    </x-button>
    <DesktopHeaderUserNavWishlistPremium
      :class="$style.wishlist"
      :theme="userNavItemsTheme"
      :is-active="isWishlistPage"
    />
    <DesktopHeaderUserNavCartPremium
      :theme="userNavItemsTheme"
      :is-active="isCartPage"
    />
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useAppStore } from 'stores/app';
import { useAuth } from 'composables/useAuth';
import { useRoute } from 'vue-router';
import { CUSTOMER_PAGE_TYPE, WISHLIST_PAGE_TYPE, CART_PAGE_TYPE } from 'blocks/common/routes/const';

import DesktopHeaderUserNavProfilePremium from 'components/desktop/header/user/nav/profile/DesktopHeaderUserNavProfilePremium.vue';
import DesktopHeaderUserNavCartPremium from 'components/desktop/header/user/nav/cart/DesktopHeaderUserNavCartPremium.vue';
import DesktopHeaderUserNavWishlistPremium from 'components/desktop/header/user/nav/wishlist/DesktopHeaderUserNavWishlistPremium.vue';
import XButton from 'blocks/x-button/x-button.vue';

const appStore = useAppStore();
const route = useRoute();
const { isLogon, showLoginModal } = useAuth();

const buttonTheme = computed(() => (appStore.isTransparentHeader ? 'secondaryTransparent' : 'label'));
const userNavItemsTheme = computed(() => (appStore.isTransparentHeader ? 'alternative' : ''));
const isCustomerPage = computed(() => route?.meta.pageType === CUSTOMER_PAGE_TYPE);
const isWishlistPage = computed(() => route?.meta.pageType === WISHLIST_PAGE_TYPE);
const isCartPage = computed(() => route?.meta.pageType === CART_PAGE_TYPE);
</script>

<style module>
  .root {
    display: flex;
    flex-flow: row nowrap;
    position: relative;
  }

  .profile {
    margin-right: 16px;
  }

  .wishlist {
    margin-right: 16px;
  }

  .button {
    min-width: 77px;
    margin-right: 16px;
  }
</style>
