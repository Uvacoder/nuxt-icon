<!-- eslint-disable vue/multi-word-component-names -->
<script setup lang="ts">
import type { IconifyIcon } from '@iconify/vue'
import { Icon as Iconify } from '@iconify/vue/dist/offline'
import { loadIcon } from '@iconify/vue'
import { useNuxtApp, useState, ref, useAppConfig, computed, watch } from '#imports'

const nuxtApp = useNuxtApp()
const nuxtIcon = useAppConfig().nuxtIcon
const props = defineProps({
  name: {
    type: String,
    required: true
  },
  size: {
    type: String,
    default: ''
  }
})
const state = useState('icons', () => ({}))
const isFetching = ref(false)
const icon = computed<IconifyIcon | null>(() => state.value?.[props.name])
const component = computed(() => nuxtApp.vueApp.component(props.name))
const sSize = computed(() => {
  const size = props.size || nuxtIcon?.size || '1em'
  if (String(Number(size)) === size) {
    return `${size}px`
  }
  return size
})

async function loadIconComponent () {
  if (component.value) {
    return
  }
  if (!state.value?.[props.name]) {
    isFetching.value = true
    state.value[props.name] = await loadIcon(props.name).catch(() => {})
    isFetching.value = false
  }
}

watch(() => props.name, loadIconComponent)

!component.value && await loadIconComponent()
</script>

<template>
  <span v-if="isFetching" class="icon" :width="sSize" :height="sSize" />
  <Iconify v-else-if="icon" :icon="icon" class="icon" :width="sSize" :height="sSize" />
  <Component :is="component" v-else-if="component" class="icon" :width="sSize" :height="sSize" />
  <span v-else class="icon" :style="{ fontSize: sSize, lineHeight: sSize, width: sSize, height: sSize }">{{ name }}</span>
</template>

<style scoped>
.icon {
  display: inline-block;
  vertical-align: middle;
}
</style>
