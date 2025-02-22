<template>
  <div :class="['bal-card', cardClasses]">
    <div v-if="!!title || $slots.header" :class="['header', headerClasses]">
      <component :is="titleTag" v-if="!!title" v-text="title" />
      <div v-if="$slots.header" class="flex-1 flex items-center">
        <slot name="header" />
      </div>
    </div>
    <div :class="['content', contentClasses]">
      <slot />
    </div>
    <div v-if="$slots.footer" :class="['footer', footerClasses]">
      <slot name="footer" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue';

export default defineComponent({
  name: 'BalCard',

  props: {
    title: { type: String, default: '' },
    titleTag: { type: String, default: 'h5' },
    square: { type: Boolean, default: false },
    noPad: { type: Boolean, default: false },
    noContentPad: { type: Boolean, default: false },
    noBorder: { type: Boolean, default: false },
    shadow: {
      type: String,
      default: '',
      validator: (val: string): boolean => {
        return ['', 'none', 'sm', 'md', 'lg'].includes(val);
      }
    }
  },

  setup(props) {
    const borderClasses = computed(() => {
      return 'border dark:border-gray-700';
    });

    const cardClasses = computed(() => {
      return {
        'rounded-lg': !props.square,
        [`shadow${props.shadow ? '-' : ''}${props.shadow}`]: true,
        [borderClasses.value]: !props.noBorder
      };
    });

    const headerClasses = computed(() => {
      return {
        'p-4 pb-3': !props.noPad
      };
    });

    const contentClasses = computed(() => {
      return {
        'p-4 pt-3': !props.noPad && !props.noContentPad
      };
    });

    const footerClasses = computed(() => {
      return {
        'rounded-b-lg': !props.square,
        'p-4 pt-3': !props.noPad
      };
    });

    return {
      cardClasses,
      contentClasses,
      headerClasses,
      footerClasses
    };
  }
});
</script>

<style scoped>
.bal-card {
  @apply bg-white dark:bg-gray-900;
}

.header {
  @apply flex items-center;
}

.footer {
  @apply flex items-center;
  @apply bg-gray-50 dark:bg-gray-800;
  @apply border-t border-gray-100 dark:border-gray-700;
}
</style>
