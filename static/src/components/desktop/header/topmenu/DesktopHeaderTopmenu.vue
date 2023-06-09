<template>
  <nav
    :class="$style.root"
    :aria-label="LABELS.TOP_MENU_ROOT"
    role="menubar"
    @mouseleave="close"
    @mouseover="isMenuHovered = true"
  >
    <DesktopHeaderTopmenuPremiumButton
      v-if="premiumButtonCategory && isPremiumTheme"
      :category="premiumButtonCategory"
      :is-on-premium-page="appStore.isPremiumPage"
      @focus="selectedCategoryName = premiumButtonCategory?.title?.text || ''"
      @click="onCategoryClick($event, premiumButtonCategory!)"
      @mouseenter="onPremiumCategoryHover(premiumButtonCategory!)"
      @mouseleave="onCategoryLeave(premiumButtonCategory!)"
    />
    <DesktopHeaderTopmenuCategory
      v-for="(category, index) in topLevelCategories"
      :key="`topmenu_main_${category.title?.text}_${appStore.shortGender}`"
      :class="[
        $style.category,
        isPremiumTheme && $style.categoryPremium,
        $experiments.test('ITPMO1608_resale') && $style.categoryDense,
      ]"
      :is-premium="appStore.isTransparentHeader"
      :category="category"
      :level="index"
      :gender-key="appStore.shortGender.toUpperCase()"
      :select-delay="selectedCategory ? 100 : 300"
      :aria-expanded="selectedCategoryName === category.title?.text && hasDropdown(category)"
      :aria-haspopup="hasDropdown(category)"
      :item-hovered="hovered"
      :is-menu-hovered="isMenuHovered"
      role="menuitem"
      @focus="selectedCategoryName = category.title?.text || ''"
      @click="onCategoryClick($event, category)"
      @mouseover="onCategoryHover(category)"
      @mouseleave="onCategoryLeave(category)"
    />
    <DesktopHeaderTopmenuDropdown
      v-if="selectedCategoryDropdown && selectedCategoryName && selectedCategoryHasDropdown"
      :key="selectedCategoryName"
      :class="$style.dropdown"
      :category="selectedCategoryDropdown"
      :disabled-classes="disabledClasses"
      :widgets-data="widgetsData"
      :meta="meta"
    />
  </nav>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted, defineAsyncComponent } from 'vue';
import { useStore } from 'vuex';
import { useRoute } from 'vue-router';
import { useExperiments } from 'composables/useExperiments';
import { useDevice } from 'composables/useDevice';
import { useApi } from 'composables/useApi';
import { useStorage } from 'composables/useStorage';
import { useI18n } from 'vue-i18n';
import * as ACTIONS from 'blocks/common/store/action-types';
import LABELS from 'blocks/common/aria-label';
import * as GETTERS from 'blocks/common/store/getter-types';
import { TopmenuCategoryClass } from 'blocks/common/topmenu';
import { logError } from 'blocks/common/utils';
import { getJust4YouRecommendations } from 'api/rec';
import isObject from 'lodash/isObject';
import cloneDeep from 'lodash/cloneDeep';
import { CookieStorageKey } from 'types/storage';
import { useAppStore } from 'stores/app';
import { useTopMenuWidgets } from 'composables/headerTopmenu/useTopMenuWidgets';
import { useTopMenuCache } from 'composables/headerTopmenu/useTopMenuCache';
import { useState } from 'composables/useState';
import { useTopMenuStats } from 'composables/statistics/useTopMenuStats';
import { useUIProgressBarStore } from 'stores/ui-progress-bar';

import { PlatformForSiteMenu } from 'api-gen/models/platform-for-site-menu';
import { SiteFlexibleMenuV1BlockContentType } from 'api-gen/models/site-flexible-menu-v1-block-content-type';

import type { Gender } from 'types/common';
import type {
  TopmenuCategory,
  TopmenuDynamicBrandsRequestParam,
  TopmenuDynamicCategoryRequestParam,
  TopmenuDynamicCategoriesResponseParams,
  TopmenuDynamicPopularBrandsResponseParams,
} from 'types/topmenu';

import DesktopHeaderTopmenuCategory from 'components/desktop/header/topmenu/DesktopHeaderTopmenuCategory.vue';
import DesktopHeaderTopmenuPremiumButton from 'components/desktop/header/topmenu/DesktopHeaderTopmenuPremiumButton.vue';

const DesktopHeaderTopmenuDropdown = defineAsyncComponent(() => import('components/desktop/header/topmenu/DesktopHeaderTopmenuDropdown.vue'));

const FIRST_OPEN_SELECT_DELAY = 300;
const ALREADY_OPEN_SELECT_DELAY = 100;

const route = useRoute();
const device = useDevice();
const experiments = useExperiments();
const storage = useStorage();
const store = useStore();
const api = useApi();
const appStore = useAppStore();
const { t } = useI18n();
const header = useState('header');

const uIProgressBarStore = useUIProgressBarStore();

const hasJust4YouProducts = ref(false);
const uuid = ref(null);
const selectedCategoryName = ref<string | null>(null);
const hovered = ref('');
const isMenuHovered = ref(false);
const meta = ref({ uuid: null });
const menuMap = ref(new Map<string, TopmenuCategory>());
const timeoutId = ref<ReturnType<typeof setTimeout> | null>(null);
// FIXME: Брать меню из стора
// @ts-ignore
const topMenu = ref<TopmenuCategory[]>(header.value?.menu_flexible?.category || []);
const premiumButtonCategory = ref<TopmenuCategory | null>(null);

const { widgetsData, fetchWidgets } = useTopMenuWidgets();
const { setMenuToStorage, getMenuFromStorage, clearMenuStorege } = useTopMenuCache();

const {
  reportTopMenuLinkClick,
  reportTopMenuPremiumButtonHover,
  reportTopMenuButtonClick,
} = useTopMenuStats();

const isLogon = computed(() => store.getters[GETTERS.USER_IS_LOGON]);

const version = computed(() => route.query.tmver as string || experiments.get('experimental_topmenu_flexible_ver')!);

const menuTopLevel = computed(() => topMenu.value.map(item => ({ class: item.class, title: item.title })));

const isPremiumTheme = computed(() => experiments.test('ITPMO918_WEB_Header') || appStore.isTransparentHeader);

const selectedCategoryDropdown = computed(() => {
  if (selectedCategoryName.value) {
    return menuMap.value.get(selectedCategoryName.value);
  }

  return null;
});

const disabledClasses = computed(() => [
  ...(experiments.test('WEB4383_Discovery_Hub') || experiments.test('WEB8462_discovery_hub_ua')
    ? [TopmenuCategoryClass.Trends, TopmenuCategoryClass.Looks]
    : [TopmenuCategoryClass.Discovery]),
  ...(experiments.test('WEB5417_just4you') && hasJust4YouProducts.value && isLogon.value
    ? []
    : [TopmenuCategoryClass.Just4You]),
  ...(isPremiumTheme.value
    ? [TopmenuCategoryClass.BrandsDefault, TopmenuCategoryClass.PremiumDefault]
    : [TopmenuCategoryClass.BrandsPremium, TopmenuCategoryClass.PremiumPremium]),
  ...(experiments.test('ITPMO1608_resale')
    ? []
    : [TopmenuCategoryClass.Resale]),
].filter(Boolean));

const selectedCategory = computed(() => {
  if (topMenu.value.length > 0) {
    return topMenu.value
      .filter(category => !disabledClasses.value.includes(category.class as TopmenuCategoryClass))
      .find(({ title }) => title?.text === selectedCategoryName.value);
  }

  return null;
});

const topLevelCategories = computed(() => {
  const menu = menuTopLevel.value
    .filter(category => !disabledClasses.value.includes(category.class as TopmenuCategoryClass))
    .filter(category => !(
      isPremiumTheme.value
        && (
          category.class === TopmenuCategoryClass.PremiumPremium
          || category.title?.text === 'Premium'
        )
    ));

  if (appStore.gender === 'kids' && isPremiumTheme.value) {
    return menu.filter(({ title }) => title?.text !== 'Premium');
  }

  return menu;
});

const clearHoverTimeout = () => {
  if (timeoutId.value) {
    clearTimeout(timeoutId.value);
    timeoutId.value = null;
  }
};

const getPremiumButtonCategory = () => {
  if (!isPremiumTheme.value) return null;

  let category = null;

  if (appStore.gender === 'kids') {
    category = menuTopLevel.value.find(({ title }) => title?.text === 'Premium');
  } else {
    category = menuTopLevel.value.find(({ class: theme }) => theme === TopmenuCategoryClass.PremiumPremium);
  }

  return category || null;
};

const isDynamicCategory = (value: SiteFlexibleMenuV1BlockContentType) => value === SiteFlexibleMenuV1BlockContentType.DynamicCategories;

const isDynamicPopularBrands = (value: SiteFlexibleMenuV1BlockContentType) => value === SiteFlexibleMenuV1BlockContentType.DynamicPopularBrands;

const close = () => {
  selectedCategoryName.value = null;
  isMenuHovered.value = false;
  hovered.value = '';
};

const getExistingCategory = () => {
  const menu = getMenuFromStorage();

  menuMap.value = menu ? new Map(menu) : new Map();
};

const createDynamicCategoriesParams = ({ blockId, dynamicCategories }: {
  blockId: number;
  dynamicCategories: TopmenuDynamicCategoriesResponseParams;
}) => ({
  block_id: blockId,
  add_all_row: !!dynamicCategories.addAllProductsRow,
  add_new_row: !!dynamicCategories.addNewProductsRow,
  category_id: dynamicCategories.categoryId,
  title: dynamicCategories.title,
  is_new: !!dynamicCategories.isNew,
  is_sale: !!dynamicCategories.isSale,
  is_outlet: !!dynamicCategories.isOutlet,
  rows_total_count: dynamicCategories.rowsTotalCount,
  rows_per_column_count: dynamicCategories.rowsPerColumnCount,
});

const createDynamicPopularBrandsParams = ({ blockId, dynamicPopularBrands }: {
  blockId: number;
  dynamicPopularBrands: TopmenuDynamicPopularBrandsResponseParams;
}) => ({
  block_id: blockId,
  category_id: dynamicPopularBrands.categoryId,
  rows_total_count: dynamicPopularBrands.rowsTotalCount,
});

const getTopMenu = async (userGender: Gender) => {
  uIProgressBarStore.runTask(async () => {
    const { data: { category } } = await api.cmsApi.cmsTopmenuFlexible({
      body: {
        aoid: storage.getCookie(CookieStorageKey.GD_AOID)?.toString() ?? '',
        // @ts-ignore
        gender: userGender === 'kids' ? 'children' : userGender,
        version: version.value,
        platform: PlatformForSiteMenu.DesktopSite,
      },
    });
    topMenu.value = category || [];
    premiumButtonCategory.value = getPremiumButtonCategory();

    if (isPremiumTheme.value) {
      fetchWidgets();
    }
  });
};

const createNewTabs = async (tab: any) => {
  const newTab: Record<string, any> = {};
  const dynamicCategoriesParams: TopmenuDynamicCategoryRequestParam[] = [];
  const dynamicPopularBrandsParams: TopmenuDynamicBrandsRequestParam[] = [];

  Object.keys(tab).forEach((key) => {
    if (isObject(tab[key]) && tab[key].groups?.data) {
      const data = tab[key].groups.data.flat();
      const categoryParams = data
        .map((item: any) => (isDynamicCategory(item.contentType) && item.dynamicCategories ? createDynamicCategoriesParams(item) : null))
        .filter(Boolean);

      const brandsParams = data
        .map((item: any) => (isDynamicPopularBrands(item.contentType) && item.dynamicPopularBrands ? createDynamicPopularBrandsParams(item) : null))
        .filter(Boolean);

      dynamicCategoriesParams.push(...categoryParams);
      dynamicPopularBrandsParams.push(...brandsParams);
    }
  });

  if (dynamicCategoriesParams.length === 0 && dynamicPopularBrandsParams.length === 0) return tab;

  try {
    const { data: { dynamicCategories } } = await api.cmsApi.cmsTopmenuFlexibleDynamicContent({
      body: {
        platform: PlatformForSiteMenu.DesktopSite,
        dynamicCategories: dynamicCategoriesParams.length > 0 ? dynamicCategoriesParams : undefined,
        dynamicPopularBrands: dynamicPopularBrandsParams.length > 0 ? dynamicPopularBrandsParams : undefined,
      },
    });

    Object.keys(tab).forEach((key) => {
      if (isObject(tab[key])) {
        const data = tab[key].groups.data.map((group: any) => group.flatMap((item: any) => {
          if (isDynamicCategory(item.contentType) && dynamicCategories) {
            return dynamicCategories
              .filter(({ blockId }) => item.blockId === blockId)
              .map(({ subcategories }) => ({
                ...item,
                items: subcategories,
              }));
          }
          return item;
        }));

        newTab[key] = {
          groups: { data },
        };
      }
    });

    return newTab;
  } catch (e) {
    logError(e);
    return tab;
  }
};

const getCategory = async () => {
  getExistingCategory();

  if (selectedCategoryName.value && !menuMap.value.has(selectedCategoryName.value)) {
    const { tabs } = selectedCategory.value?.tabsWrapper || {};

    if (tabs) {
      const newTabs = await Promise.all(tabs.map(tab => createNewTabs(tab)));

      const newDropdownCategory = cloneDeep(selectedCategory.value);
      newDropdownCategory!.tabsWrapper!.tabs = newTabs;

      menuMap.value = new Map(menuMap.value.set(selectedCategoryName.value, newDropdownCategory!));
    } else {
      menuMap.value = new Map(menuMap.value.set(selectedCategoryName.value, cloneDeep(selectedCategory.value!)));
    }

    setMenuToStorage(menuMap.value);
  }
};

const onCategoryHover = ({ title }: TopmenuCategory) => {
  hovered.value = title?.text || '';

  if (!timeoutId.value) {
    timeoutId.value = setTimeout(() => {
      if (hovered.value) {
        selectedCategoryName.value = hovered.value;
        uIProgressBarStore.runTask(getCategory);
      } else {
        hovered.value = '';
      }
      clearHoverTimeout();
    },
    selectedCategoryName.value ? ALREADY_OPEN_SELECT_DELAY : FIRST_OPEN_SELECT_DELAY);
  }
};

const onPremiumCategoryHover = (category: TopmenuCategory) => {
  onCategoryHover(category);
  reportTopMenuPremiumButtonHover();
};

const hasDropdown = (category: TopmenuCategory) => {
  const isIdeasTabWithMenu = category.title?.text === t('ideas') && Boolean(category.hoverMenu?.header);

  return isIdeasTabWithMenu || category.tabsWrapper?.tabs?.length;
};

const selectedCategoryHasDropdown = computed(() => {
  if (isMenuHovered.value && selectedCategoryDropdown.value) {
    return hasDropdown(selectedCategoryDropdown.value);
  }

  return false;
});

const onCategoryLeave = ({ title }: TopmenuCategory) => {
  if (hovered.value === title?.text) {
    hovered.value = '';
  }
  clearHoverTimeout();
};

const onCategoryClick = (event: Event, { title }: TopmenuCategory) => {
  if (isPremiumTheme.value && title?.text === 'Premium') {
    reportTopMenuButtonClick();
  } else {
    reportTopMenuLinkClick({
      element: title?.text ?? '',
      parentCategory: title?.text ?? '',
      gender: appStore.gender,
      previousUrl: route.fullPath,
      url: title?.url ?? '',
    });
  }

  if (device.isTouch) {
    event.preventDefault();
  }
};

watch(() => route.fullPath, close);

watch(() => appStore.gender, async (gender) => {
  await getTopMenu(gender);
  clearMenuStorege();
  menuMap.value.clear();
});

onMounted(async () => {
  if (isPremiumTheme.value) fetchWidgets();

  premiumButtonCategory.value = getPremiumButtonCategory();

  await getTopMenu(appStore.gender);
  if (experiments.test('WEB5417_just4you') && isLogon.value) {
    try {
      const { total = 0, uuid: just4YouUuid } = await getJust4YouRecommendations({
        gender: appStore.gender,
        limit: 0,
      });
      hasJust4YouProducts.value = total > 0;
      uuid.value = just4YouUuid;
    } catch (e) {
      logError(e);
    }
  }
});

onUnmounted(clearHoverTimeout);
</script>

<style module>
  .root {
    display: flex;
    flex-shrink: 0;
  }

  .category {
    margin-right: 12px;
    font-size: 16px;
    line-height: 50px;
  }

  .categoryPremium {
    margin-right: 24px;
  }

  .categoryDense {
    margin-right: 8px;
  }

  .categoryPremium.categoryDense {
    margin-right: 16px;
  }

  .category:last-child {
    margin-right: 0;
  }

  @media (--wide-viewport) {
    .categoryDense:not(:last-child) {
      margin-right: 12px;
    }

    .categoryPremium:not(:last-child) {
      margin-right: 32px;
    }
  }
</style>
