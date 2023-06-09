<template>
  <header :class="$style.root">
    <DesktopHeaderTop
      :class="stretch ? $style.topRowStretch : $style.topRow"
      :stretch="stretch"
    />
    <div :class="appStore.isTransparentHeader && $style.headerWrapperAlt">
      <div
        :class="[
          {
            [$style.middleRow]:!stretch,
            [$style.widthContainer]: !stretch,
            [$style.widthContainerStretch]: stretch,
            [$style.middleRowStretch]: stretch,
            [$style.topPosition]: isSearchOpen,
            'DT2268_dobroshrift': $experiments.test('DT2268_dobroshrift')
          }
        ]"
      >
        <Logo :class="[$style.logo, appStore.isTransparentHeader && $style.logoAlt]" />
        <div :class="$style.navWrapper">
          <DesktopHeaderGenders :class="$style.genders" />
          <DesktopHeaderSearchInputWrapper
            :class="$style.searchInput"
            :stretch="stretch"
            @update:search="isSearchOpen = $event"
          />
          <client-only>
            <DesktopHeaderUserNavPremium :class="$style.userNav" />
          </client-only>
        </div>
      </div>
      <div :class="stretch ? $style.bottomRowStretch : $style.bottomRow">
        <div
          :class="[
            {
              [$style.menuContainer]: !stretch,
              [$style.widthContainer]: !stretch,
              [$style.widthContainerStretch]: stretch,
              'top-menu_discovery': $experiments.test('WEB4383_Discovery_Hub')
            }]"
        >
          <div :class="$style.menuWrapper">
            <DesktopHeaderTopmenu />
          </div>
        </div>
      </div>
    </div>
    <DesktopHeaderBannerPayment
      v-if="$experiments.test('WEB_4537_prepayment_banner')"
      :class="appStore.isTransparentHeader && $style.bannerPaymentAlt"
    />
  </header>
</template>

<script setup lang="ts">
import { computed, defineAsyncComponent, ref } from 'vue';
import { useAppStore } from 'stores/app';
import { useExperiments } from 'composables/useExperiments';
import { useRoute } from 'vue-router';

import Logo from 'components/logo/Logo.vue';
import DesktopHeaderUserNavPremium from 'components/desktop/header/user/nav/DesktopHeaderUserNavPremium.vue';
import DesktopHeaderGenders from 'components/desktop/header/genders/DesktopHeaderGenders.vue';
import DesktopHeaderTop from 'components/desktop/header/DesktopHeaderTop.vue';
import DesktopHeaderTopmenu from 'components/desktop/header/topmenu/DesktopHeaderTopmenu.vue';

const stretch = computed(() => experiments.test('WEB_10572') && route.name === 'product');
const DesktopHeaderSearchInputWrapper = defineAsyncComponent(() => import('components/desktop/header/search/DesktopHeaderSearchInputWrapper.vue'));
const DesktopHeaderBannerPayment = defineAsyncComponent(() => import('components/desktop/header/banner/DesktopHeaderBannerPayment.vue'));

const experiments = useExperiments();
const route = useRoute();

defineProps({
  stretch: {
    type: Boolean,
  },
});

const appStore = useAppStore();

const isSearchOpen = ref(false);
</script>

<style module>
.root {
  background-color: var(--backgroundColor);
}
.headerWrapperAlt {
  position: relative;
}

.headerWrapperAlt::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0.5;
  background-color: var(--labelColor);
  z-index: 1;
}

.middleRow,
.middleRowStretch {
  min-height: 90px;
  position: relative;
  z-index: var(--zIndexPremium);
  display: flex;
  justify-content: space-between;
}

.middleRow.topPosition,
.middleRowStretch.topPosition {
  z-index: 34;
}

.logo {
  margin-top: 30px;
}

.logoAlt {
  filter: invert(1);
}

.navWrapper {
  flex-grow: 1;
  display: flex;
  align-items: center;
  margin-left: 52px;
  justify-content: space-between;
}

.genders {
  position: relative;
  left: 0;
  top: 4px;
}

.searchInput {
  flex-grow: 1;
}

.userNav {
  position: relative;
  margin-left: 24px;
  font: var(--fontSizeSubLine);
  display: flex;
  align-items: center;
}

.bottomRow,
.bottomRowStretch {
  position: relative;
  color: var(--labelColor);
  z-index: var(--zIndexPremium);
  box-shadow: var(--shadowPremiumHeader);
}

.menuWrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-flow: row nowrap;
}

.widthContainer {
  max-width: var(--desktopLayout);
  margin: 0 auto;
}

.widthContainerStretch {
  min-width: var(--desktopLayout);
  margin: 0 auto;
  padding: 0 36px;
  max-width: var(--maxDesktopLayout);
}

.topRow {
  position: relative;
  z-index: 36;
}

.topRow,
.middleRow,
.bottomRow {
  padding: 0 16px;
}

.topRowStretch {
  padding: 0 36px;
}

.bannerPaymentAlt {
  position: absolute;
  z-index: 1;
}

@media (--wide-viewport) {
  .middleRow {
    padding: 0 20px;
  }

  .widthContainer {
    max-width: var(--maxLayout);
  }

  .navWrapper {
    margin-left: 82px;
  }
}
</style>
