<template>
  <div :class="[$style.dropdown, isWeb8401 && $style.popoverWEB8401]">
    <template v-if="isWeb8401">
      <DesktopHeaderUserProfile :class="$style.userProfile" />
      <DesktopUserNavLoyalty :class="$style.userLoyalty" />
    </template>
    <x-user-nav-loyalty v-else />
    <x-menu-link
      v-for="(link, index) in menuStoreLinks"
      :key="`d-header-user-nav-profile_link_${index}`"
      :link="link"
      :class="$style.item"
      source="profile_hover_menu"
    />
    <div :class="$style.hr"></div>
    <a href="/customer/account/logout/" :class="$style.item">
      {{ $t('logout') }}
    </a>
    <a
      v-if="$experiments.test('1974_total_logout')"
      :class="$style.item"
      @click="logoutAll"
    >
      {{ $t('logout_all') }}
    </a>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useExperiments } from 'composables/useExperiments';
import { useMenuStore } from 'stores/menu';
import { useUIAuthStore } from 'stores/ui-auth';
import { useLazyAsyncData } from 'composables/asyncData';
import { useNuxtApp } from 'composables/useNuxtApp';
import { useResetSessionsModalStats } from 'composables/statistics/useResetSessionsModalStats';

import XUserNavLoyalty from 'blocks/x-user-nav-loyalty/x-user-nav-loyalty.vue';
import DesktopHeaderUserProfile from 'components/desktop/header/user/nav/profile/DesktopHeaderUserProfile.vue';
import DesktopUserNavLoyalty from 'components/desktop/header/user/nav/loyalty/DesktopUserNavLoyalty.vue';
import XMenuLink from 'blocks/x-menu-link/x-menu-link.vue';

const emit = defineEmits(['close']);

const experiments = useExperiments();
const nuxt = useNuxtApp();
const menuStore = useMenuStore();
const UIAuthStore = useUIAuthStore();
const { reportOpenModalClick } = useResetSessionsModalStats();

const isWeb8401 = experiments.get('web_8401_lp_redesign') === 'show';

const menuStoreLinks = computed(() => (isWeb8401
  ? menuStore.popupLinksWEB8401
  : menuStore.popupLinks));

const logoutAll = () => {
  reportOpenModalClick();
  UIAuthStore.openLogoutModal();
  emit('close');
};

useLazyAsyncData('menu-counters', async () => {
  await menuStore.fetchAllCounters(nuxt);
});
</script>

<style module>
  .item {
    cursor: pointer;
    display: block;
    line-height: 1.38;
    padding: 7px 24px;
    font-weight: 400;
    color: var(--labelColor);
  }

  .popoverWEB8401 .item {
    padding: 8px 16px;
    font: var(--fontSizeSubLine);
  }

  .item:hover {
    background-color: var(--secondaryColor);
  }

  .dropdown {
    background: var(--backgroundColor);
    padding: 7px 0;
    width: 285px;
  }

  .popoverWEB8401.dropdown {
    box-sizing: border-box;
    padding: 16px 0 8px 0;
    width: 372px;
  }

  .hr {
    overflow: hidden;
    height: 1px;
    background: var(--separatorColor);
    margin: 7px 24px;
  }

  .popoverWEB8401 .hr {
    margin: 8px 16px;
  }

  .userProfile {
    margin: 0 16px 16px 16px;
  }

  .userLoyalty {
    margin: 16px 16px 9px 16px;
  }
</style>
