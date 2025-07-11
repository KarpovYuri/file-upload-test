<template>
  <button
    type="button"
    class="base-button"
    :disabled
  >
    <slot>Кнопка</slot>
  </button>
</template>

<script setup lang="ts">
import './BaseButton.css'

interface BaseButtonProps {
  disabled?: boolean
}

withDefaults(defineProps<BaseButtonProps>(), {
  disabled: false,
})
</script>
