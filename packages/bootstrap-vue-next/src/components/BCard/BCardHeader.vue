<template>
  <BCardHeadFoot class="card-header" v-bind="props">
    <slot />
  </BCardHeadFoot>
</template>

<script setup lang="ts">
import type {ColorVariant, TextColorVariant} from '../../types'
import BCardHeadFoot from './BCardHeadFoot.vue'

const props = withDefaults(
  defineProps<{
    text?: string
    bgVariant?: ColorVariant | null
    borderVariant?: ColorVariant | null
    html?: string
    tag?: string
    textVariant?: TextColorVariant | null
  }>(),
  {
    tag: 'div',
    text: undefined,
    bgVariant: undefined,
    borderVariant: undefined,
    html: undefined,
    textVariant: undefined,
  }
)

defineSlots<{
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  default?: (props: Record<string, never>) => any
}>()
</script>
