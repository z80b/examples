<template>
  <header :class="$style.root">
    <DesktopHeaderTop
      :class="stretch ? $style.topRowStretch : $style.topRow"
      :stretch="stretch"
    />
    <div
      :class="[
        stretch ? $style.middleRowStretch : $style.middleRow,
        stretch ? $style.widthContainerStretch : $style.widthContainer,
        $experiments.test('DT2268_dobroshrift') && 'DT2268_dobroshrift']"
    >
      <Logo :class="$style.logo" />
      <client-only>
        <div :class="stretch ? $style.navWrapperStretch : $style.navWrapper">
          <DesktopHeaderUserNav />
        </div>
      </client-only>
      <div :class="stretch ? $style.genderWrapperStretch : $style.genderWrapper">
        <DesktopHeaderGenders />
      </div>
    </div>
    <div :class="[stretch ? $style.bottomRowStretch : $style.bottomRow, isSearchOpen && $style.topPosition]">
      <div
        :class="[
          $style.menuContainer,
          stretch ? $style.widthContainerStretch : $style.widthContainer,
          $experiments.test('WEB4383_Discovery_Hub') && 'top-menu_discovery']"
      >
        <div :class="$style.menuWrapper">
          <DesktopHeaderTopmenu />
          <client-only>
            <DesktopHeaderSearchInputWrapper @update:search="isSearchOpen = $event" />
          </client-only>
        </div>
      </div>
      <DesktopHeaderBannerPayment v-if="$experiments.test('WEB_4537_prepayment_banner')" />
    </div>
  </header>
</template>

<script setup lang="ts">
import { computed, defineAsyncComponent, ref } from 'vue';
import { useExperiments } from 'composables/useExperiments';
import { useRoute } from 'vue-router';

import Logo from 'components/logo/Logo.vue';
import DesktopHeaderUserNav from 'components/desktop/header/user/nav/DesktopHeaderUserNav.vue';
import DesktopHeaderGenders from 'components/desktop/header/genders/DesktopHeaderGenders.vue';
import DesktopHeaderTop from 'components/desktop/header/DesktopHeaderTop.vue';
import DesktopHeaderTopmenu from 'components/desktop/header/topmenu/DesktopHeaderTopmenu.vue';

const experiments = useExperiments();
const route = useRoute();

const stretch = computed(() => experiments.test('WEB_10572') && route.name === 'product');
const DesktopHeaderSearchInputWrapper = defineAsyncComponent(() => import('components/desktop/header/search/DesktopHeaderSearchInputWrapper.vue'));
const DesktopHeaderBannerPayment = defineAsyncComponent(() => import('components/desktop/header/banner/DesktopHeaderBannerPayment.vue'));

defineProps({
  stretch: {
    type: Boolean,
    default: false,
  },
});

const isSearchOpen = ref(false);
</script>

<style module>
.root {
  background-color: var(--backgroundColor);
}

.topRow {
  z-index: 36;
}

.middleRow {
  min-height: 90px;
  text-align: center;
  z-index: 32;
}

.topPosition {
  z-index: 34;
}

.bottomRow {
  color: var(--labelColor);
  z-index: 32;
}

.bottomRow.topPosition {
  z-index: 34;
}

.middleRow,
.topRow,
.bottomRow {
  padding: 0 16px;
  position: relative;
  box-sizing: border-box;
}

.topRowStretch {
  padding: 0 36px;
  position: relative;
  box-sizing: border-box;
}

.middleRowStretch {
  min-height: 90px;
  text-align: center;
  z-index: 32;
  position: relative;
}

.bottomRowStretch {
  padding: 0;
  position: relative;
  box-sizing: border-box;
  color: var(--labelColor);
  z-index: 32;
}

.bottomRowStretch.topPosition {
  z-index: 34;
}

.navWrapper {
  position: absolute;
  right: 20px;
  top: 34px;
}

.navWrapperStretch {
  position: absolute;
  right: 36px;
  top: 34px;
}

.genderWrapper {
  position: absolute;
  left: 0;
  top: 36px;
}

.genderWrapperStretch {
  position: absolute;
  left: 36px;
  top: 36px;
}

/* FIXME: Перебиваем стили x-link. Использовать flex вместо position */
.root .logo {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.menuWrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-flow: row nowrap;
}

.widthContainer {
  max-width: 960px;
  margin: 0 auto;
}

.widthContainerStretch {
  min-width: var(--desktopLayout);
  margin: 0 auto;
  padding: 0 36px;
  max-width: var(--maxDesktopLayout);
}

@media (--wide-viewport) {
  .middleRow {
    padding: 0 20px;
  }

  .widthContainer {
    max-width: 1200px;
  }
}
</style>
