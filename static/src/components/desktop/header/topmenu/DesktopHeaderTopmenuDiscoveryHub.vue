<template>
  <div :class="$style.root">
    <x-link
      :href="href"
      :class="$style.textWrapper"
      themes="link"
    >
      <div :class="$style.title">
        {{ title }}
      </div>
      <transition name="fade">
        <div :key="itemText" :class="$style.subTitle">
          {{ itemText }}
        </div>
      </transition>
    </x-link>
    <div :class="$style.imageWrapper">
      <a :href="href">
        <transition name="fade">
          <div :class="$style.imageHover">
            <img
              :key="itemImage"
              :class="$style.image"
              :src="itemImage"
            />
          </div>
        </transition>
      </a>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onBeforeMount, onUnmounted } from 'vue';

import type { PropType } from 'vue';
import type { TopmenuCategory, TopmenuDiscoverySlide } from 'types/topmenu';

import XLink from 'blocks/x-link/x-link.vue';

const CHANGE_ITEM_DELAY = 2500;
const IMAGE_PATH = '//a.lmcdn.ru/files/cms/';

const props = defineProps({
  category: {
    type: Object as PropType<TopmenuCategory>,
    required: true,
  },
});

const itemText = ref('');
const itemImage = ref('');
const itemLength = ref(0);
const itemCurrent = ref(1);
const title = ref('');
const slides = ref<TopmenuDiscoverySlide[]>([]);

const href = computed(() => props.category.title?.url);

const getImage = (number: number) => IMAGE_PATH + slides.value[number].image;

const changeItem = () => {
  itemText.value = slides.value[itemCurrent.value].text;
  itemImage.value = getImage(itemCurrent.value);

  if (itemCurrent.value === itemLength.value) {
    itemCurrent.value = 0;
  } else {
    itemCurrent.value += 1;
  }
};

const interval = setInterval(changeItem, CHANGE_ITEM_DELAY);

onBeforeMount(() => {
  slides.value = props.category.hoverMenu?.slides ?? [];
  title.value = props.category.hoverMenu?.header ?? '';

  itemText.value = slides.value[0].text;
  itemImage.value = getImage(0);
  itemLength.value = slides.value.length - 1;
});

onUnmounted(() => {
  clearInterval(interval);
});
</script>

<style>
  .fade-enter-active,
  .fade-leave-active {
    transition: ease opacity 0.3s;
  }

  .fade-enter,
  .fade-leave-active {
    opacity: 0;
  }
</style>

<style module>
  .root {
    display: flex;
    margin-bottom: -6px;
  }

  .textWrapper {
    width: 320px;
  }

  .imageWrapper {
    margin-left: auto;
    order: 2;
    width: 468px;
    height: 312px;
  }

  .title {
    font: var(--fontSizeAccident);
    margin-top: 24px;
  }

  .subTitle {
    font: var(--fontSizeHeadLine);
    width: 320px;
    margin-top: 24px;
    position: absolute;
  }

  .image {
    display: block;
    position: absolute;
  }

  .imageHover {
    display: block;
    position: absolute;
    width: 468px;
    height: 312px;
  }

  .imageHover::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: var(--secondaryBorderColor);
    opacity: 0;
    transition: ease opacity 0.3s;
  }

  .imageHover:hover::after {
    opacity: 0.2;
  }

</style>
