<template>
  <div
    v-if="(remainTime > 0) && isLogon"
    :class="$style.paymentWrapper">
    <div :class="$style.payment">
      <div :class="$style.text">{{ $t('online_payment_failed') }} {{ remainTime }} {{ $t('within_minutes__pluralize', remainTime) }}</div>
      <x-link
        :class="$style.link"
        themes="label"
        href="/sales/order/history/">{{ $t('go_order') }}
      </x-link>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onBeforeUnmount } from 'vue';
import { getRemainingTime } from 'blocks/common/utils/date';
import { useStore } from 'vuex';
import { useState } from 'composables/useState';
import { CookieStorageKey } from 'types/storage';
import { useStorage } from 'composables/useStorage';

import XLink from 'blocks/x-link/x-link.vue';

const storage = useStorage();

const store = useStore();
const stateRequest: any = useState('request');

const remainTime = ref(0);

const isLogon = computed(() => store.state.user.isLogon);

const decreaseRemainTime = () => {
  remainTime.value -= 1;
};

const expiresTimeCookie = storage.getCookie(CookieStorageKey.ORDER_EXPIRES) as number;
const currentTime = new Date(stateRequest.value.timestamp * 1000).getTime();
const expiresTime = expiresTimeCookie ? new Date(expiresTimeCookie).getTime() : Date.now();

remainTime.value = getRemainingTime(expiresTime, currentTime).minutes;

const decreaseRemainTimeId = setInterval(decreaseRemainTime, 60000);

onBeforeUnmount(() => {
  clearInterval(decreaseRemainTimeId);
});
</script>

<style module>
  .paymentWrapper {
    background: var(--additionalPalePinkColor);
    height: 40px;
    width: 100%;
    font: var(--fontSizeSubLine);
  }

  .payment {
    width: 960px;
    height: 100%;
    margin: 0 auto;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .link {
    text-decoration: underline;
    font: var(--fontSize);
  }

  @media (--wide-viewport) {
    .payment {
      width: var(--maxLayout);
    }
  }
</style>
