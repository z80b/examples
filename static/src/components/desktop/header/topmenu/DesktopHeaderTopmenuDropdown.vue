<template>
  <div :class="$style.root">
    <div :class="$style.container">
      <x-tabs-default
        v-if="clickableTabs.length > 0"
        v-model:active-tab="active"
        :tabs="clickableTabs.map(tab => ({
          id: tab.target,
          title: tab.text,
          tabClass: {
            [$style.tab]: true,
            [$style.tabActive]: active === tab.target,
          },
        }))"
        :class="$style.tabs"
        :header-class="$style.tabsHeader"
      >
        <template
          v-for="({ target }, index) in clickableTabs"
          #[target!]
          :key="target"
        >
          <DesktopHeaderPremiumTopmenuDropdownTab
            v-if="isPremiumTabComponent"
            :tab="tabs[index]"
            :title="title"
            :category="category"
            :allow-empty-title="false"
            :disabled-classes="disabledClasses"
            :brands-list="brandsList"
            :categories-list="categoriesList"
            :meta="meta"
          />
          <DesktopHeaderTopmenuDropdownTab
            v-else
            :tab="tabs[index]"
            :title="title"
            :category="category"
            :allow-empty-title="false"
            :disabled-classes="disabledClasses"
            :meta="meta"
          />
        </template>
      </x-tabs-default>
      <template v-else-if="tabs.length">
        <div v-if="isPremiumTabComponent">
          <DesktopHeaderPremiumTopmenuDropdownTab
            v-for="(tab, index) in tabs"
            :key="index"
            :tab="tab"
            :title="title"
            :category="category"
            :disabled-classes="disabledClasses"
            :brands-list="brandsList"
            :categories-list="categoriesList"
            :meta="meta"
          />
        </div>
        <div v-else>
          <DesktopHeaderTopmenuDropdownTab
            v-for="(tab, index) in tabs"
            :key="index"
            :tab="tab"
            :title="title"
            :category="category"
            :disabled-classes="disabledClasses"
            :meta="meta"
          />
        </div>
      </template>
      <DesktopHeaderTopmenuDiscoveryHub v-else-if="isDiscoveryHub" :category="category" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineAsyncComponent, ref, computed, watch, onMounted } from 'vue';
import { useI18n } from 'vue-i18n';
import { useAppStore } from 'stores/app';
import { TopmenuCategoryClass } from 'blocks/common/topmenu';
import { reduceToStringByKeys } from 'blocks/common/statistics/utils';
import { useExperiments } from 'composables/useExperiments';
import { useTopMenuStats } from 'composables/statistics/useTopMenuStats';

import type { PropType } from 'vue';
import type { WidgetsData, DisabledClasses, TopmenuCategory } from 'types/topmenu';

import DesktopHeaderTopmenuDropdownTab from 'components/desktop/header/topmenu/DesktopHeaderTopmenuDropdownTab.vue';
import DesktopHeaderPremiumTopmenuDropdownTab from 'components/desktop/header/topmenu/DesktopHeaderPremiumTopmenuDropdownTab.vue';
import DesktopHeaderTopmenuDiscoveryHub from 'components/desktop/header/topmenu/DesktopHeaderTopmenuDiscoveryHub.vue';

const XTabsDefault = defineAsyncComponent(() => import('blocks/x-tabs-default/x-tabs-default.vue'));

const props = defineProps({
  category: {
    type: Object as PropType<TopmenuCategory>,
    required: true,
  },
  disabledClasses: {
    type: Array as PropType<DisabledClasses>,
    default: () => [],
  },
  widgetsData: {
    type: Object as PropType<WidgetsData>,
    default: () => ({}),
  },
  meta: {
    type: Object,
    default: () => ({}),
  },
});

const { t } = useI18n();
const experiments = useExperiments();
const appStore = useAppStore();

const { reportTopMenuShowDropdown, reportTopMenuDropdownTabClick } = useTopMenuStats();

const activeTabTarget = ref('');

const title = computed(() => props.category?.title?.text);
const clickableTabs = computed(() => {
  const tabs = props.category?.tabsWrapper?.triggers?.top?.trigger ?? [];

  return tabs.filter(({ text }) => Boolean(text));
});

const tabs = computed(() => props.category?.tabsWrapper?.tabs ?? []);

const active = computed({
  get() {
    if (activeTabTarget.value) {
      return activeTabTarget.value;
    }

    if (clickableTabs.value.length) {
      return clickableTabs.value[0].target;
    }

    return 'not_found';
  },
  set(value) {
    activeTabTarget.value = value || '';
  },
});

const isDiscoveryHub = computed(() => props.category?.title?.text === t('ideas'));
const isPremiumHeaderEnabled = computed(() => experiments.test('ITPMO918_WEB_Header') || appStore.isTransparentHeader);
const isPremiumTabComponent = computed(() => {
  if (isPremiumHeaderEnabled.value) {
    return [TopmenuCategoryClass.BrandsPremium, TopmenuCategoryClass.PremiumPremium].includes(props.category.class as TopmenuCategoryClass);
  }

  return false;
});

const brandsList = computed(() => {
  if (isPremiumTabComponent.value) {
    switch (props.category.class) {
      case TopmenuCategoryClass.PremiumPremium:
        return props.widgetsData?.premiumBrandsTilesList;
      case TopmenuCategoryClass.BrandsPremium:
        return props.widgetsData?.brandsBrandsTilesList;
      default:
        return null;
    }
  }
  return null;
});

const categoriesList = computed(() => {
  if (isPremiumTabComponent.value && props.category.class === TopmenuCategoryClass.PremiumPremium) {
    return props.widgetsData?.premiumCategoryTilesList;
  }

  return null;
});

const hasBanner = () => {
  if (clickableTabs.value.length) {
    const currentTabIndex = clickableTabs.value.map(tab => tab.target).indexOf(active.value);
    const currentTab = tabs.value[currentTabIndex];
    const columns = currentTab.top?.groups?.data?.flat() || [];

    return columns.some(item => Boolean(item.banner));
  }

  if (tabs.value.length) {
    const columns = tabs.value[0].top?.groups?.data?.flat() || [];

    return columns.some(item => Boolean(item.banner));
  }

  return false;
};

watch(active, (newTarget) => {
  const column = clickableTabs.value.find(({ target }) => target === newTarget);

  reportTopMenuDropdownTabClick({
    tab: column?.text || '',
    category: props.category?.title?.text ?? '',
    banner: Number(hasBanner()),
  });
});

onMounted(() => {
  reportTopMenuShowDropdown({
    category: props.category?.title?.text ?? '',
    banner: Number(hasBanner()),
    tab: clickableTabs.value ? reduceToStringByKeys(['text'], ';', clickableTabs.value) : '',
  });
});
</script>

<style module>
  .root {
    z-index: 32;
    padding: 15px 16px 30px;
    position: absolute;
    top: 50px;
    right: 0;
    left: 0;
    width: 100%;
    background-color: var(--backgroundColor);
    box-shadow: 0 6px 8px 0 rgba(0, 0, 0, 0.16);
    box-sizing: border-box;
  }

  .container {
    margin: 0 auto;
    max-width: 960px;
  }

  .tabsHeader {
    margin-bottom: 10px;
    border-bottom: var(--thinBorder);
    line-height: 20px;
  }

  /* custom дизайн tabs переопределяем стандартные стили vue tabs */
  /* stylelint-disable declaration-no-important */
  .tab {
    font: var(--fontSize) !important;
    line-height: normal;
    margin-right: 30px !important;
    color: var(--labelColor) !important;
  }

  .tabActive {
    border-bottom: 3px solid var(--secondaryBorderColor);
    padding-bottom: 12px !important;
    font-weight: bold !important;
  }

  @media (--wide-viewport) {
    .container {
      max-width: var(--maxLayout);
    }
  }
</style>
