<template>
  <svg viewBox="0 0 40 40">
    <defs>
      <mask
        id="mask"
        width="40"
        height="40"
      >
        <rect
          fill="black"
          width="100%"
          height="100%"
        />
        <path
          fill-rule="evenodd"
          clip-rule="evenodd"
          d="M20 40C4.969 40 0 35.046 0 20.0621C0 5.07765 4.969 0 20 0C35.0316 0 40 5.07765 40 20.0621C40 35.046 35.0316 40 20 40Z"
          fill="white"
        />
      </mask>
    </defs>

    <template v-if="isFallback">
      <path
        fill-rule="evenodd"
        clip-rule="evenodd"
        d="M20 40C4.969 40 0 35.046 0 20.0621C0 5.07765 4.969 0 20 0C35.0316 0 40 5.07765 40 20.0621C40 35.046 35.0316 40 20 40Z"
        :fill="fallbackColor"
      />

      <text
        x="20"
        y="20"
        font-size="18"
        dominant-baseline="central"
        text-anchor="middle"
        fill="var(--badgeForegroundColor)"
      >
        {{ fallback }}
      </text>
    </template>

    <image
      v-else
      height="40"
      width="40"
      :href="avatar"
      preserveAspectRatio="xMinYMin slice"
      mask="url(#mask)"
    />
  </svg>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const fallbackColorsArray = [
  '--additionalGoldColor',
  '--additionalMintColor',
  '--additionalPaleBlueColor',
  '--additionalPalePinkColor',
  '--additionalBlue20FlatColor',
  '--additionalBlueColor',
];

const props = defineProps({
  avatar: {
    type: String,
    default: '',
  },
  fallback: {
    type: String,
    required: true,
  },
});

const getRandomItem = (items: string[] = []) => items[Math.floor(Math.random() * items.length)];
const fallbackColor = `var(${getRandomItem(fallbackColorsArray)})`;

const isFallback = computed(() => !props.avatar);
</script>
