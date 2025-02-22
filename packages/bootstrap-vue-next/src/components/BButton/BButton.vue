<template>
  <component
    :is="computedTag"
    class="btn"
    :class="computedClasses"
    :aria-disabled="nonStandardTag ? disabledBoolean : null"
    :aria-pressed="isToggle ? pressedBoolean : null"
    :autocomplete="isToggle ? 'off' : null"
    :disabled="isButton ? disabledBoolean : null"
    :href="href"
    :rel="computedLink ? rel : null"
    :role="nonStandardTag || computedLink ? 'button' : null"
    :target="computedLink ? target : null"
    :type="isButton ? type : null"
    :to="!isButton ? to : null"
    :append="computedLink ? append : null"
    :active-class="isBLink ? activeClass : null"
    :event="isBLink ? event : null"
    :replace="isBLink ? replace : null"
    :router-component-name="isBLink ? routerComponentName : null"
    :router-tag="isBLink ? routerTag : null"
    @click="clicked"
  >
    <template v-if="loadingBoolean">
      <slot name="loading">
        <template v-if="!loadingFillBoolean">
          {{ loadingText }}
        </template>
        <slot name="loading-spinner">
          <BSpinner :small="size !== 'lg'" :label="loadingFillBoolean ? loadingText : undefined" />
        </slot>
      </slot>
    </template>
    <template v-else>
      <slot />
    </template>
  </component>
</template>

<script setup lang="ts">
import {computed} from 'vue'
import BSpinner from '../BSpinner.vue'
import {useBLinkHelper, useBooleanish} from '../../composables'
import type {BLinkProps, Booleanish, ButtonType, ButtonVariant, Size} from '../../types'
import BLink from '../BLink/BLink.vue'
import {useVModel} from '@vueuse/core'

defineSlots<{
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  'default'?: (props: Record<string, never>) => any
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  'loading'?: (props: Record<string, never>) => any
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  'loading-spinner'?: (props: Record<string, never>) => any
}>()

const props = withDefaults(
  defineProps<
    {
      pill?: Booleanish
      pressed?: Booleanish
      size?: Size
      squared?: Booleanish
      tag?: string
      type?: ButtonType
      variant?: ButtonVariant | null
      loading?: Booleanish
      loadingFill?: Booleanish
      block?: Booleanish
      loadingText?: string
    } & Omit<BLinkProps, 'variant'>
  >(),
  {
    pill: false,
    pressed: undefined,
    size: 'md',
    squared: false,
    tag: 'button',
    type: 'button',
    variant: 'secondary',
    loading: false,
    loadingFill: false,
    block: false,
    loadingText: 'Loading...',
    // Link props
    active: false,
    activeClass: undefined,
    append: false,
    disabled: false,
    event: 'click',
    href: undefined,
    // noPrefetch: {type: [Boolean, String] as PropType<Booleanish>, default: false},
    // prefetch: {type: [Boolean, String] as PropType<Booleanish>, default: null},
    rel: undefined,
    replace: false,
    routerComponentName: 'router-link',
    routerTag: 'a',
    target: '_self',
    to: undefined,
    opacity: undefined,
    opacityHover: undefined,
    underlineVariant: null,
    underlineOffset: undefined,
    underlineOffsetHover: undefined,
    underlineOpacity: undefined,
    underlineOpacityHover: undefined,
    icon: false,
    // End link props
  }
)

const emit = defineEmits<{
  'click': [value: MouseEvent]
  'update:pressed': [value: boolean]
}>()

const pressedValue = useVModel(props, 'pressed', emit)

const activeBoolean = useBooleanish(() => props.active)
const blockBoolean = useBooleanish(() => props.block)
const disabledBoolean = useBooleanish(() => props.disabled)
const pillBoolean = useBooleanish(() => props.pill)
const pressedBoolean = useBooleanish(() => props.pressed)
const squaredBoolean = useBooleanish(() => props.squared)
const loadingBoolean = useBooleanish(() => props.loading)
const loadingFillBoolean = useBooleanish(() => props.loadingFill)

const {computedLink} = useBLinkHelper(props)

const isToggle = computed(() => typeof pressedBoolean.value === 'boolean')
const isButton = computed(
  () => props.tag === 'button' && props.href === undefined && props.to === undefined
)
const isBLink = computed(() => props.to !== undefined)
const nonStandardTag = computed(() => (props.href !== undefined ? false : !isButton.value))

const computedClasses = computed(() => [
  [`btn-${props.size}`],
  {
    [`btn-${props.variant}`]: props.variant !== null,
    'btn-block': blockBoolean.value,
    'active': activeBoolean.value || pressedBoolean.value,
    'rounded-pill': pillBoolean.value,
    'rounded-0': squaredBoolean.value,
    'disabled': disabledBoolean.value,
  },
])

const computedTag = computed(() => (isBLink.value ? BLink : props.href ? 'a' : props.tag))

const clicked = (e: MouseEvent): void => {
  if (disabledBoolean.value) {
    e.preventDefault()
    e.stopPropagation()
    return
  }
  emit('click', e)
  if (isToggle.value) {
    pressedValue.value = !pressedBoolean.value
  }
}
</script>
