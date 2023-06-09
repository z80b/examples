<template>
  <div
    :class="$style.root"
    @keydown.enter.exact="onSearchSubmit"
    @keyup.esc.exact="onSearchReset"
    @keydown.down.exact.prevent="onKeyDown"
    @keydown.up.exact.prevent="onKeyUp"
    @mouseenter="onFormMouseenter"
    @mouseleave="isHighlightOn = false"
  >
    <input
      ref="inputElement"
      v-model="inputText"
      autocomplete="off"
      type="text"
      maxlength="128"
      :placeholder="placeholder"
      :class="[$style.input, {[$style.inputShown]: isShown}]"
      @focus="onFocus"
      @blur="onBlur"
      @input="onInput"
    />
    <x-button
      theme="primaryFilled"
      :size="32"
      :class="$style.button"
      :intrinsic-width="false"
      @click="onButtonClick"
    >
      <x-icon
        :type="XIcon.types.searchWhite"
        :size="24"
        :class="$style.icon"
      />
    </x-button>
    <div
      v-if="isShowSuggests && (suggest.length || recents.length)"
      :class="[$style.suggest, $style.shadow]"
      @mouseleave="selectedIndex = -1"
    >
      <template v-if="(routeQuery === rawQuery && recents.length === 0) || suggest.length">
        <div
          v-for="({ text }, index) in suggest"
          :key="index"
          :class="[
            $style.suggestItem,
            {[$style.suggestItemSelected]: selectedIndex === index},
          ]"
          @mouseenter="selectSuggestItem(index)"
          @mousedown="onSearchSubmit"
          v-html="getSuggestItemHtml(text)"
        ></div>
      </template>
      <template v-else>
        <DesktopSearchRecents
          :is-old-dsk-search="true"
          :selected-index="selectedIndex"
          @mouseenter="(index) => selectSuggestItem(index)"
          @mousedown="onSearchSubmit"
        />
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onBeforeMount, onMounted, nextTick } from 'vue';
import { useSearch } from 'composables/useSearch';
import { useDevice } from 'composables/useDevice';
import { useRoute } from 'vue-router';
import { useSearchStats } from 'composables/statistics/useSearchStats';

import XButton from 'blocks/x-button/x-button.vue';
import XIcon from 'blocks/x-icon/x-icon.vue';
import DesktopSearchRecents from 'components/desktop/header/search/DesktopHeaderSearchRecents.vue';

const device = useDevice();
const {
  suggest,
  requestSuggest,
  recents,
  recentsToShow,
  rawQuery,
  query,
  routeQuery,
  inputText,
  placeholder,
  inputElement,
  isInputFocused,
  isShowSuggests,
  selectedIndex,
  isHighlightOn,
  getSuggestItemHtml,
  selectSuggestItem,
  reportShowSuggest,
  onFormMouseenter,
  onInput,
  onKeyDown,
  onKeyUp,
  onPasteClipboardValue,
  onSearchReset,
  onSearchSubmit,
} = useSearch();

const { reportSearchShow, reportSearchStart } = useSearchStats();
const route = useRoute();

const emit = defineEmits(['update:search']);

const clicked = ref(false);

const isShown = computed(() => clicked.value || device.isWide);

const onButtonClick = () => {
  if (query.value && query.value !== routeQuery.value) {
    onSearchSubmit();
    return;
  }
  clicked.value = true;
  nextTick(() => inputElement.value?.focus());
};

const onBlur = () => {
  if (query.value && clicked.value) {
    clicked.value = false;
  }
  isInputFocused.value = false;
  isShowSuggests.value = false;
};

const onFocus = () => {
  isInputFocused.value = true;
  isShowSuggests.value = true;

  reportSearchStart({ elementType: 'search_bar' });

  if (recentsToShow.value.length) {
    reportShowSuggest('previous');
  }

  if (routeQuery.value === rawQuery.value) {
    requestSuggest();
  }
};

watch(route, (currentRoute) => {
  rawQuery.value = currentRoute.query?.q as string ?? '';
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
  inputElement.value?.addEventListener('paste', onPasteClipboardValue);
});
</script>

<style module>
  .root {
    float: right;
    position: relative;
    overflow: visible;
    width: 57px;
    height: 30px;
    z-index: 32;
  }

  .button {
    width: 57px;
  }

  .icon {
    margin: auto;
  }

  .input {
    display: block;
    position: absolute;
    width: 0;
    top: 0;
    right: 55px;
    overflow: hidden;
    cursor: pointer;
    box-sizing: border-box;
    opacity: 0;
    visibility: hidden;
    height: 32px;
    line-height: 32px;
    padding: 0 8px;
    border: 1px solid var(--secondaryLabelColor);
    border-right: 0;
    outline: none;
  }

  .input:focus {
    border: 1px solid var(--primaryColor);
    border-right: 0;
  }

  .inputShown {
    width: 236px;
    cursor: default;
    opacity: 1;
    visibility: visible;
  }

  .suggest {
    position: absolute;
    font: var(--fontSize);
    display: flex;
    flex-direction: column;
    background-color: var(--badgeForegroundColor);
    overflow-y: auto;
    overflow-x: hidden;
    width: 290px;
    top: calc(100% + 4px);
    right: 0;
    padding: 5px 0;
    border-radius: 4px;
  }

  .suggest::after {
    content: "";
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: var(--primaryColor);
    opacity: 0.21;
    z-index: -1;
  }

  .suggestItem {
    padding: 8px 20px;
    background-color: var(--backgroundColor);
    cursor: pointer;
    transition: all 0.2s linear;
  }

  .suggestItem b {
    font-weight: 400;
    color: var(--secondaryLabelColor);
  }

  .suggestItemSelected,
  .suggestItem:focus {
    background-color: var(--secondaryColor);
  }

  .shadow {
    box-shadow: var(--shadowBox);
  }
</style>
