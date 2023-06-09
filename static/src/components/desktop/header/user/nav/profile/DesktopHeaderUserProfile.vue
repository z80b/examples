<template>
  <x-link href="/customer/account/" :class="$style.root">
    <DesktopHeaderUserProfileAvatar
      :avatar="user.avatarUrl"
      :fallback="avatarFallback"
      :class="$style.avatar"
    />
    <div :class="$style.infoWrapper">
      <span :class="$style.name">{{ user.fullName }}</span>
      <span>{{ user.email }}</span>
    </div>
  </x-link>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useStore } from 'vuex';
import * as GETTERS from 'blocks/common/store/getter-types';

import XLink from 'blocks/x-link/x-link.vue';
import DesktopHeaderUserProfileAvatar from 'components/desktop/header/user/nav/profile/DesktopHeaderUserProfileAvatar.vue';

const store = useStore();

const user = computed(() => store.getters[GETTERS.USER_DETAILS]);

const avatarFallback = computed(() => {
  const { firstName, lastName } = user.value;
  let initials = '';
  if (firstName) initials += firstName[0];
  if (lastName) initials += lastName[0];
  return initials.toUpperCase();
});
</script>

<style module>
.root {
  display: flex;
  align-items: center;
}

.avatar {
  width: 40px;
  height: 40px;
  margin-right: 8px;
}

.infoWrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
  overflow: hidden;
  font: var(--fontSize);
  color: var(--secondaryLabelColor);
}

.infoWrapper span {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.name {
  margin-bottom: 4px;
  font: var(--fontSizeSubLine);
  color: var(--labelColor);
}
</style>
