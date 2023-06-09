<template>
  <div
    ref="searchElement"
    :class="[$style.root, isWide && $style.normalWidth]"
    @keydown.enter.exact="onSearchSubmit"
    @keyup.esc.exact="onSearchReset"
    @keydown.down.exact.prevent="onKeyDown"
    @keydown.up.exact.prevent="onKeyUp"
  >
    <div :class="[isOpenSuggestsBlock && $style.background]" @click="onCloseSearch"></div>
    <div :class="[$style.inputWrapper, widthClass]">
      <x-search-input
        ref="inputElement"
        v-model="inputText"
        :is-focused="isInputFocused"
        :placeholder="$t('search')"
        :is-show-suffix="isShowSuffix && !isInputFocused"
        :suffix="paginationStore.totalItems!"
        @focus="onFocusInput"
        @paste-text="onPasteClipboardValue"
        @update:model-value="onInputQuery"
      />
      <x-transition-fade mode="out-in" speed="fast">
        <x-search-suggests
          v-if="isOpenSuggestsBlock"
          :list="suggestList"
          :is-recent="isShowRecentSuggests"
          :query="rawQuery"
          :is-highlight="isHighlightOn"
          :is-allow-to-remove="isShowRecentSuggests"
          :selected-index="selectedIndex"
          @hover="onSuggestsHover"
          @select="onSelectSuggest"
          @remove="onRemoveSuggest"
        />
      </x-transition-fade>
    </div>
    <x-button
      theme="primaryFilled"
      :size="40"
      :class="[$style.button, (isWide || isInputFocused) && $style.squadRadiusButton]"
      :intrinsic-width="false"
      :aria-label="$t('search')"
      @click="onButtonClick"
    >
      <span>{{ $t('find') }}</span>
    </x-button>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, useCssModule, watch, onBeforeMount, onMounted, onBeforeUnmount, nextTick } from 'vue';
import { useSearch } from 'composables/useSearch';
import { useRoute } from 'vue-router';
import { useSearchStats } from 'composables/statistics/useSearchStats';
import { useExperiments } from 'composables/useExperiments';
import { usePaginationStore } from 'stores/pagination';
import { useUIProgressBarStore } from 'stores/ui-progress-bar';

import XTransitionFade from 'blocks/x-transition-fade/x-transition-fade.vue';
import XButton from 'blocks/x-button/x-button.vue';
import XSearchInput from 'blocks/x-search-input/x-search-input.vue';
import XSearchSuggests from 'blocks/x-search-suggests/x-search-suggests.vue';

defineProps({
  isShowSuffix: {
    type: Boolean,
    default: false,
  },
});

const searchElement = ref<HTMLElement | null>(null);
const inputElement = ref<InstanceType<typeof XSearchInput> | null>(null);

const emit = defineEmits(['update:search']);

const style = useCssModule();
const experiments = useExperiments();
const {
  reportSearchShow,
  reportSearchRemoveRecent,
  reportSearchStart,
  reportSearchShowSuggest,
} = useSearchStats();
const {
  suggest,
  requestSuggest,
  recents,
  recentsToShow,
  removeRecentEntry,
  rawQuery,
  query,
  isInputFocused,
  isShowSuggests,
  selectedIndex,
  inputText,
  routeQuery,
  isHighlightOn,
  onPasteClipboardValue,
  onSearchSubmit,
  onSearchReset,
  onKeyDown,
  onKeyUp,
} = useSearch();
const route = useRoute();

const isWide = ref(false);

const paginationStore = usePaginationStore();
const uIProgressBarStore = useUIProgressBarStore();

const isShowRecentSuggests = computed(() => rawQuery.value.trim().length === 0 || suggest.value.length === 0);
const suggestList = computed(() => (isShowRecentSuggests.value ? recentsToShow.value : suggest.value));

const widthClass = computed(() => {
  if (!isWide.value && isInputFocused.value) {
    return style.wideWidth;
  }
  return style.normalWidth;
});

const isOpenSuggestsBlock = computed(() => isShowSuggests.value && (suggestList.value.length > 0 || experiments.test('RNDDA-4582_aggregated_brands')));

const checkSize = () => {
  const parentNode = searchElement.value?.parentNode as HTMLElement;
  isWide.value = parentNode.clientWidth >= 210;
  isInputFocused.value = false;
  isShowSuggests.value = false;
};

const onSuggestsHover = ({ index }: { index: number}) => {
  isHighlightOn.value = false;
  selectedIndex.value = index;
};

const onRemoveSuggest = ({ text, index }: { text: string; index: number }) => {
  removeRecentEntry(text);

  reportSearchRemoveRecent({
    elementType: 'search_bar',
    query: text,
    suggest: {
      list: recents.value,
      type: 'recent_suggest',
    },
    newSuggest: {
      list: recents.value,
    },
    item: `${index}:${text}`,
  });

  rawQuery.value = '';
};

const onInputQuery = () => {
  isHighlightOn.value = true;
  isShowSuggests.value = true;
  selectedIndex.value = -1;
  requestSuggest();
};

const onSelectSuggest = ({ text }: { text: string}) => {
  isInputFocused.value = false;
  isShowSuggests.value = false;
  isHighlightOn.value = false;
  rawQuery.value = text;
  onSearchSubmit();
};

const onFocusInput = () => {
  isInputFocused.value = true;
  isShowSuggests.value = true;

  reportSearchStart({ elementType: 'search_bar' });

  if (isShowRecentSuggests.value) {
    reportSearchShowSuggest({
      query: query.value,
      suggest: {
        type: 'previous',
        list: recents.value,
      },
    });
  }
};

const onButtonClick = () => {
  if (!isInputFocused.value) {
    inputElement.value?.inputElement.setFocus();
    onFocusInput();
    return;
  }

  if (isWide.value) {
    onSearchSubmit();
  }
};

const onCloseSearch = () => {
  isInputFocused.value = false;
  isShowSuggests.value = false;
  isHighlightOn.value = false;
};

watch(() => uIProgressBarStore.isActive, (value) => {
  if (!value) {
    checkSize();
  }
});

watch(route, (currentRoute) => {
  rawQuery.value = currentRoute.query?.q as string ?? '';
});

watch(suggest, (value) => {
  if (value.length > 0) {
    reportSearchShowSuggest({
      query: query.value,
      suggest: {
        type: 'text',
        list: value,
      },
    });
  }
});

watch(isInputFocused, (value) => {
  emit('update:search', value);
});

onBeforeMount(() => {
  if (routeQuery.value) {
    rawQuery.value = routeQuery.value;
  }
});

onMounted(() => {
  reportSearchShow({ elementType: 'search_bar' });
  window.addEventListener('resize', checkSize);
  nextTick(checkSize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', checkSize);
});
</script>

<style module>
  .root {
    margin: 0 0 0 auto;
    position: relative;
    width: 70px;
    height: 40px;
    transition: width 0s;
    z-index: 32;
  }

  .background {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: var(--primaryColor);
    opacity: 0.21;
    z-index: -1;
  }

  .inputWrapper {
    transition: width 0.4s ease-out;
  }

  .button {
    width: 70px;
    z-index: 2;
  }

  .squadRadiusButton {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }

  .inputWrapper,
  .button {
    position: absolute;
    top: 0;
    right: 0;
    height: 100%;
  }

  .normalWidth {
    width: 100%;
  }

  .wideWidth {
    width: 550px;
  }
</style>
