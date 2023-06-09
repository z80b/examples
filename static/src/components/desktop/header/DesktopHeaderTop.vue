<template>
  <div
    :class="[
      stretch ? $style.rootStretch : $style.root,
      {[$style.black]: !$experiments.test('WEB6335_Header_USP') || isPremiumHeaderTheme},
    ]">
    <div :class="stretch ? $style.wrapperStretch : $style.wrapper">
      <div
        :class="[
          $style.left,
          appStore.isTransparentHeader && $style.leftPremium,
        ]"
      >
        <DesktopGeoChooser
          :theme="geoChooserTheme"
          :class="$style.geo"
          class="js-header-geo-wrapper"
        />
        <x-no-translate>
          <DesktopGoogleTranslate
            v-if="!isDevelopment && $experiments.test('WEB6217')"
            :theme="noTranslateComponentsTheme"
            :class="$style.translate"
          />
          <DesktopLocaleSelector
            v-if="isDevelopment && $experiments.test('WEB6217')"
            :theme="noTranslateComponentsTheme"
            :class="$style.translate"
          />
        </x-no-translate>
      </div>
      <div
        :class="[
          $style.right,
          appStore.isTransparentHeader && $style.rightPremium,
        ]">
        <x-banner-json
          v-if="isPremiumHeaderTheme && appStore.isTransparentHeader"
          banner-slot="premium_usp"
          :class="$style.uspPremium"
        />
        <x-banner-json
          v-for="index in 3"
          v-else
          :key="`header-usp-${index}`"
          is-mobapi
          :banner-slot="`js_header_usp${index}`"
          :class="$style.usp"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useExperiments } from 'composables/useExperiments';
import { useAppStore } from 'stores/app';
import { isDevelopment as isDevelopmentFunc } from 'blocks/common/utils';

import DesktopGeoChooser, { GeoChooserTheme } from 'components/desktop/GeoChooser/DesktopGeoChooser.vue';
import XBannerJson from 'blocks/x-banner-json/x-banner-json.vue';
import XNoTranslate from 'blocks/x-no-translate/x-no-translate.vue';
import DesktopLocaleSelector from 'components/desktop/LocaleSelector/DesktopLocaleSelector.vue';
import DesktopGoogleTranslate from 'components/desktop/GoogleTranslate/DesktopGoogleTranslate.vue';

const experiments = useExperiments();

const appStore = useAppStore();

defineProps({
  stretch: {
    type: Boolean,
    required: false,
  },
});

const isPremiumHeaderTheme = computed(() => experiments.test('ITPMO918_WEB_Header') || appStore.isTransparentHeader);

const noTranslateComponentsTheme: GeoChooserTheme = experiments.test('WEB6335_Header_USP') ? 'grey' : 'black';
const geoChooserTheme: GeoChooserTheme = isPremiumHeaderTheme.value ? 'premium' : noTranslateComponentsTheme;

const isDevelopment = isDevelopmentFunc();
</script>

<style module>
  .root {
    background-color: var(--secondaryBackgroundColor);
    height: 40px;
  }

  .rootStretch {
    background-color: var(--secondaryBackgroundColor);
    height: 40px;
  }

  .root.black,
  .rootStretch.black {
    background-color: var(--labelColor);
  }

  .wrapper {
    display: flex;
    justify-content: space-between;
    width: 100%;
    height: 100%;
    margin: 0 auto;
  }

  .wrapperStretch {
    display: flex;
    justify-content: space-between;
    height: 100%;
    margin: 0 auto;
    min-width: var(--desktopLayout);
    max-width: var(--maxDesktopLayout);
  }

  .left,
  .right {
    display: flex;
  }

  .leftPremium {
    min-width: 35%;
  }

  .rightPremium {
    flex-grow: 1;
  }

  .geo {
    padding: 0 12px;
    margin: 0 0 0 -14px;
    cursor: pointer;
  }

  .translate {
    height: 40px;
    position: relative;
  }

  .usp {
    z-index: 100;
  }

  .uspPremium {
    z-index: 100;
    height: 100%;
  }

  @media (--desktop-viewport) {
    .wrapper {
      width: var(--desktopLayout);
    }

    .right {
      margin-right: -20px;
    }
  }

  @media (--wide-viewport) {
    .wrapper {
      width: var(--maxLayout);
    }
  }
</style>
