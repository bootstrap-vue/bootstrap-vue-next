<template>
  <component
    :is="tagComputed"
    class="list-group-item"
    :class="computedClasses"
    :aria-current="activeBoolean ? true : undefined"
    :aria-disabled="disabledBoolean ? true : undefined"
    :target="isLink ? target : undefined"
    :href="!buttonBoolean ? href : undefined"
    :to="!buttonBoolean ? to : undefined"
    v-bind="computedAttrs"
  >
    <slot />
  </component>
</template>

<script setup lang="ts">
import {computed, inject, useAttrs} from 'vue'
import type {BLinkProps, Booleanish} from '../../types'
import {useBLinkHelper, useBooleanish} from '../../composables'
import BLink from '../BLink/BLink.vue'
import {listGroupInjectionKey} from '../../utils'

const props = withDefaults(
  defineProps<
    {
      action?: Booleanish
      button?: Booleanish
      tag?: string
    } & Omit<BLinkProps, 'event' | 'routerTag'>
  >(),
  {
    action: false,
    button: false,
    tag: 'div',
    // Link props
    active: false,

    activeClass: undefined,
    append: false,
    href: undefined,
    // noPrefetch: {type: [Boolean, String] as PropType<Booleanish>, default: false},
    // prefetch: {type: [Boolean, String] as PropType<Booleanish>, default: null},
    rel: undefined,
    replace: false,
    disabled: false,
    routerComponentName: 'router-link',
    target: '_self',
    to: undefined,
    opacity: undefined,
    opacityHover: undefined,
    underlineVariant: null,
    underlineOffset: undefined,
    underlineOffsetHover: undefined,
    underlineOpacity: undefined,
    underlineOpacityHover: undefined,
    variant: null,
  }
)

defineSlots<{
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  default?: (props: Record<string, never>) => any
}>()

const attrs = useAttrs()

const parentData = inject(listGroupInjectionKey, null)

const actionBoolean = useBooleanish(() => props.action)
const activeBoolean = useBooleanish(() => props.active)
const buttonBoolean = useBooleanish(() => props.button)
const disabledBoolean = useBooleanish(() => props.disabled)

const {computedLink} = useBLinkHelper(props)

const isLink = computed(() => !buttonBoolean.value && computedLink.value)
const tagComputed = computed(() =>
  parentData?.numbered.value
    ? 'li'
    : buttonBoolean.value
    ? 'button'
    : !isLink.value
    ? props.tag
    : BLink
)

const isAction = computed(
  () =>
    actionBoolean.value ||
    isLink.value ||
    buttonBoolean.value ||
    ['a', 'router-link', 'button', 'b-link'].includes(props.tag)
)

const computedClasses = computed(() => ({
  [`list-group-item-${props.variant}`]: props.variant !== null,
  'list-group-item-action': isAction.value,
  'active': activeBoolean.value,
  'disabled': disabledBoolean.value,
}))

const computedAttrs = computed(() => {
  const localAttrs = {} as {type?: string; disabled?: boolean}
  if (buttonBoolean.value) {
    if (!attrs || !attrs.type) {
      // Add a type for button is one not provided in passed attributes
      localAttrs.type = 'button'
    }
    if (disabledBoolean.value) {
      // Set disabled attribute if button and disabled
      localAttrs.disabled = true
    }
  }
  return localAttrs
})
</script>
