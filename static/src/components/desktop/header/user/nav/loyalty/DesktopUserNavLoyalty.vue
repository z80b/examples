<template>
  <div :class="$style.root">
    <div :class="$style.card">
      <x-link href="/customer/account/loyalty_program/" :class="$style.clubCard">
        <div :class="$style.discountWrapper">
          <x-icon :type="XIcon.types.logoClub" />
          <span>{{ $t('my_discount') }}</span>
          <span>{{ loyaltyStore.discount }}%</span>
        </div>
        <div :class="$style.backgroundImage"></div>
        <div :class="$style.backgroundColor"></div>
      </x-link>

      <div v-if="loyaltyStore.actions.length" :class="$style.offers">
        <x-link
          v-for="{ siteUrl, discounts } of loyaltyStore.actions"
          :key="siteUrl"
          :href="siteUrl"
          :class="$style.offer"
        >
          <x-badges
            :badges="[{ type: 'discount', text: discounts[0].label }]"
            :class="$style.offerBadge"
          />
          {{ discounts[0].title }}
        </x-link>
      </div>
    </div>

    <div v-if="isShowLinks" :class="$style.links">
      <DesktopUserNavLoyaltyLink
        v-if="$experiments.get('ITPMO_988_my_promocodes') === 'show' || isEmployee"
        :href="couponsLink"
        :counter="counters?.coupons || 0"
        :icon="XIcon.types.loyaltyPromocodes"
        :active-icon="XIcon.types.loyaltyPromocodesHovered"
      >
        {{ $t('promocodes') }}
      </DesktopUserNavLoyaltyLink>
      <DesktopUserNavLoyaltyLink
        v-if="$experiments.test('WEB7012_gamification2')"
        href="/customer/account/offers/?sitelink=profile_hover_menu"
        :counter="counters?.offers || 0"
        :icon="XIcon.types.loyaltyGifts"
        :active-icon="XIcon.types.loyaltyGiftsHovered"
      >
        {{ $t('gifts') }}
      </DesktopUserNavLoyaltyLink>
      <DesktopUserNavLoyaltyLink
        v-if="$experiments.get('WEB-7671') === 'show'"
        href="/customer/account/achievements/"
        :counter="counters?.achievements || 0"
        :icon="XIcon.types.loyaltyAchievements"
        :active-icon="XIcon.types.loyaltyAchievementsHovered"
      >
        {{ $t('achievements') }}
      </DesktopUserNavLoyaltyLink>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import { computed, watch } from 'vue';
import { useApi } from 'composables/useApi';
import { useStore } from 'vuex';
import { useLoyaltyStore } from 'stores/loyalty';
import { useLoyaltyStats } from 'composables/statistics/useLoyaltyStats';
import { useExperiments } from 'composables/useExperiments';
import * as GETTERS from 'blocks/common/store/getter-types';
import { InlineObject3NamesEnum } from 'api-gen';
import { useLazyAsyncData } from 'composables/asyncData';

import XLink from 'blocks/x-link/x-link.vue';
import XBadges from 'blocks/x-badges/x-badges.vue';
import XIcon from 'blocks/x-icon/x-icon.vue';
import DesktopUserNavLoyaltyLink from 'components/desktop/header/user/nav/loyalty/DesktopUserNavLoyaltyLink.vue';

const { t } = useI18n();
const api = useApi();
const store = useStore();
const loyaltyStore = useLoyaltyStore();
const { reportLoyaltyBlockShow } = useLoyaltyStats();
const experiments = useExperiments();

const isEmployee = computed(() => store.getters[GETTERS.USER_IS_EMPLOYEE]);

const isShowLinks = computed(
  () => isEmployee.value ||
    experiments.get('ITPMO_988_my_promocodes') === 'show' ||
    experiments.test('WEB7012_gamification2') ||
    experiments.get('WEB-7671') === 'show',
);

const couponsLink = computed(() => (experiments.get('ITPMO_988_my_promocodes') === 'show'
  ? '/customer/account/coupons/'
  : '/customer/account/corporate-coupons/'));

const { data: counters } = useLazyAsyncData('loyalty-counters', async () => {
  const { data } = await api.badges.badgesGetCounters({
    body: {
      names: [InlineObject3NamesEnum.Achievement, InlineObject3NamesEnum.Coupon, InlineObject3NamesEnum.Offer],
    },
  });
  return data;
});

watch(counters, (countersValue) => {
  reportLoyaltyBlockShow({
    counters: countersValue,
    promo: `[${t('my_discount')}:${loyaltyStore.discount}]`,
  });
});
</script>

<style module>
.root {
  display: flex;
  flex-direction: column;
}

.card {
  display: flex;
  flex-direction: column;
  border-radius: 4px;
  overflow: hidden;
  box-shadow: var(--shadowBox);
}

.clubCard {
  position: relative;
  box-sizing: border-box;
  overflow: hidden;
  min-height: 104px;
  padding: 16px;
}

.backgroundImage {
  position: absolute;
  top: -3px;
  right: -5px;
  height: 114px;
  width: 85px;
  transition: transform 0.6s;
  background: url("./loyalty-card-background.png");
  background-size: contain;
}

.backgroundColor {
  position: absolute;
  top: 62px;
  right: -36px;
  width: 130px;
  height: 100px;
  transform: rotate(-7deg);
  transition: transform 0.5s;
  background: var(--additionalPalePinkColor);
}

.clubCard:hover .backgroundImage {
  transform: translate(-4px, -5px);
}

.clubCard:hover .backgroundColor {
  transform: rotate(-7deg) translate(-15px, -15px);
}

.discountWrapper {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  font: var(--fontSizeHeadLine);
  color: var(--labelColor);
}

.offers {
  padding: 16px;
  border-top: var(--thinBorder);
}

.offer {
  display: flex;
  align-items: baseline;
  font: var(--fontSize);
  color: var(--labelColor);
}

.offer + .offer {
  margin-top: 4px;
}

.offerBadge {
  margin-right: 16px;
  min-width: 52px;
}

.links {
  display: flex;
  justify-content: center;
  padding-bottom: 16px;
  margin-top: 16px;
  border-bottom: var(--thinBorder);
}
</style>
