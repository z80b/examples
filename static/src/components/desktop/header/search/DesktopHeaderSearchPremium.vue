<template>
  <div
    :class="[
      $style.root,
      { [$style.rootStretch]: stretch },
      { [$style.rootSecondary]: appStore.isTransparentHeader },
      { [$style.rootInputFocused] :isInputFocused }
    ]"
    @keydown.enter.exact="onSearchSubmit"
    @keyup.esc.exact="onSearchReset"
    @keydown.down.exact.prevent="onKeyDown"
    @keydown.up.exact.prevent="onKeyUp"
    @mouseenter="onFormMouseenter"
    @mouseleave="isHighlightOn = false"
  >
    <div :class="[displaySuggestBlock && $style.background]" @click="onCloseSearch"></div>
    <input
      ref="inputElement"
      v-model="inputText"
      autocomplete="off"
      type="text"
      maxlength="128"
      :placeholder="placeholder"
      :class="$style.input"
      @focus="onFocus"
      @blur="onBlur"
      @input="onInput"
    />
    <x-button
      theme="borderlessPrimary"
      :size="40"
      :class="$style.button"
      :intrinsic-width="false"
      @click="onButtonClick"
    >
      <x-icon
        :type="XIcon.types.searchMagnifyingGlass"
        :size="24"
        :class="$style.icon"
      />
    </x-button>
    <x-search-input-suffix
      :is-show-suffix="isShowSuffix"
      :query="rawQuery"
      :suffix="paginationStore.totalItems ?? ''"
      is-premium
    />
    <x-transition-fade mode="out-in" speed="fast">
      <div
        v-if="displaySuggestBlock"
        :class="[$style.suggest, $style.shadow]"
        @mouseleave="selectedIndex = -1"
      >
        <template v-if="(routeQuery === rawQuery && recents.length === 0) || suggest.length">
          <div
            v-for="({ text }, index) in suggest"
            :key="`suggest-item-${index}`"
            :class="[
              $style.suggestItem,
              {[$style.suggestItemSelected]: selectedIndex === index},
            ]"
            @mouseenter="() => selectSuggestItem(index)"
            @mousedown="onSearchSubmit"
            v-html="getSuggestItemHtml(text)"
          ></div>
        </template>
        <DesktopHeaderSearchRecents
          v-else
          :selected-index="selectedIndex"
          @mouseenter="(index) => selectSuggestItem(index)"
          @mousedown="onSearchSubmit"
        />
      </div>
    </x-transition-fade>
  </div>
</template>

<script setup lang="ts">
import { computed, watch, onBeforeMount, onMounted, nextTick } from 'vue';
import { useSearchStats } from 'composables/statistics/useSearchStats';
import { useSearch } from 'composables/useSearch';
import { useRoute } from 'vue-router';
import { usePaginationStore } from 'stores/pagination';
import { useAppStore } from 'stores/app';

import XButton from 'blocks/x-button/x-button.vue';
import XIcon from 'blocks/x-icon/x-icon.vue';
import DesktopHeaderSearchRecents from 'components/desktop/header/search/DesktopHeaderSearchRecents.vue';
import XTransitionFade from 'blocks/x-transition-fade/x-transition-fade.vue';
import XSearchInputSuffix from 'blocks/x-search-input/x-search-input-suffix.vue';

const {
  suggest,
  recents,
  recentsToShow,
  rawQuery,
  query,
  routeQuery,
  placeholder,
  inputElement,
  inputText,
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

defineProps({
  isShowSuffix: {
    type: Boolean,
    default: false,
  },
  stretch: {
    type: Boolean,
    default: false,
  },
});

const appStore = useAppStore();
const paginationStore = usePaginationStore();

const displaySuggestBlock = computed(() => isShowSuggests.value && (suggest.value.length || recents.value.length));

const onButtonClick = () => (query.value ? onSearchSubmit() : nextTick(() => inputElement.value?.focus()));

const onBlur = () => {
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
};

const onCloseSearch = () => {
  isInputFocused.value = false;
  isShowSuggests.value = false;
};

watch(isInputFocused, (value) => {
  emit('update:search', value);
});

watch(route, (currentRoute) => {
  rawQuery.value = currentRoute.query?.q as string ?? '';
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
    position: relative;
    display: flex;
    height: 40px;
    align-items: center;
    border-radius: 4px;
    border: 1px solid var(--secondaryLabelColor);
    transition: border-color var(--animationDuration);
  }

  .rootStretch {
    float: right;
    width: 100%;
    min-width: 265px;
    max-width: 349px;
  }

  .root:hover,
  .root.rootInputFocused {
    border-color: var(--labelColor);
  }

  .button {
    width: 50px;
    background-color: var(--backgroundColor);

    /* Обнуляем border-radius из стилей x-button. Нужны только с правой стороны. */
    border-radius: 0;
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
  }

  .icon {
    margin: auto;
    transition: filter var(--animationDuration);
  }

  .root:hover .icon,
  .root.rootInputFocused .icon {
    filter: brightness(0);
  }

  .input {
    display: block;
    flex-grow: 1;
    overflow: hidden;
    cursor: pointer;
    border: none;
    border-top-left-radius: inherit;
    border-bottom-left-radius: inherit;
    box-sizing: border-box;
    height: 100%;
    padding: 10px;
    outline: none;
  }

  /* Более специфичный селектор для переопределения размера шрифта, заданного в defaults.ccs для инпутов */
  .root .input {
    font: var(--fontSizeSubLine);
  }

  .root .input::placeholder {
    font: var(--fontSizeSubLine);
  }

  .rootSecondary {
    border-color: var(--overlayLightColor50);
  }

  .rootSecondary:hover,
  .rootSecondary.rootInputFocused {
    border-color: var(--backgroundColor);
  }

  /* Добавлена специфичность с помощью .root, иначе линтер ругается на no-descending-specificity */
  .root.rootSecondary .icon {
    filter: brightness(0) invert(1);
    opacity: 0.5;
    transition: opacity var(--animationDuration);
  }

  .root.rootSecondary:hover .icon,
  .root.rootSecondary.rootInputFocused .icon {
    opacity: 1;
  }

  .rootSecondary .input {
    background-color: transparent;
    color: var(--backgroundColor);
  }

  .rootSecondary .input::placeholder {
    color: var(--overlayLightColor50);
    transition: color var(--animationDuration);
  }

  .rootSecondary:hover .input::placeholder {
    color: var(--backgroundColor);
  }

  .rootSecondary .button {
    background-color: transparent;
  }

  .suggest {
    position: absolute;
    display: flex;
    flex-direction: column;
    font: var(--fontSizeSubLine);
    background-color: var(--badgeForegroundColor);
    overflow-y: auto;
    overflow-x: hidden;
    left: 0;
    right: 0;
    top: calc(100% + 4px);
    padding: 6px 0;
    border-radius: 4px;
  }

  .suggestItem {
    padding: 5px 10px;
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
    background-color: var(--secondaryBackgroundColor);
  }

  .shadow {
    box-shadow: var(--shadowBox);
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
</style>
