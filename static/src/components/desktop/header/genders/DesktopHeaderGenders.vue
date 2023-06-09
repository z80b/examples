<template>
  <nav :class="style.root" role="menubar">
    <x-link
      v-for="gender in genders"
      :key="gender.id"
      :class="getClasses(gender.id)"
      :href="getLink(gender.href)"
      :themes="getThemes(gender.id)"
      :data-genders="gender.id"
      :data-active="isActive(gender.id)"
      :aria-label="gender.label"
      role="menuitem"
      @click="appStore.rawGender = gender.id"
    >
      {{ $t(gender.title) }}
    </x-link>
  </nav>
</template>

<script setup lang="ts">
import { useCssModule } from 'vue';
import { useRoute } from 'vue-router';
import { useAppStore } from 'stores/app';
import { useGenders } from 'composables/useGenders';

import XLink from 'blocks/x-link/x-link.vue';

const style = useCssModule();
const route = useRoute();
const appStore = useAppStore();
const { genders } = useGenders();

const isActive = (targetGender: string) => {
  if (targetGender === 'children' && appStore.gender === 'kids') {
    return true;
  }
  return appStore.gender === targetGender;
};

const getThemes = (targetGender: string) => {
  if (appStore.isTransparentHeader) {
    return isActive(targetGender) ? 'badgeForeground' : 'premiumSecondary';
  }
  return isActive(targetGender) ? 'label' : 'secondaryLabel';
};

const getClasses = (targetGender: string) => [
  style.link,
  { [style.linkActive]: isActive(targetGender) },
  { [style.linkActiveSecondary]: isActive(targetGender) && appStore.isTransparentHeader },
];

const getLink = (link: string) => {
  let resultLink = link;
  if (route.query.site_topmenu_type) {
    resultLink = `${resultLink}&site_topmenu_type=${route.query.site_topmenu_type}`;
  }
  if (route?.query?.tmver) {
    resultLink = `${resultLink}&tmver=${route.query.tmver}`;
  }
  return resultLink;
};
</script>

<style module>
  .root {
    font: var(--fontSizeSubLine);
    display: flex;
    justify-content: space-between;
    width: 254px;
  }

  .currentPage {
    display: none;
  }

  .link {
    display: inline-block;
    padding-bottom: 5px;
    margin-right: 20px;
  }

  .linkActive {
    border-bottom: var(--middleSecondaryBorder);
  }

  .linkActiveSecondary {
    border-color: var(--badgeForegroundColor);
  }
</style>
