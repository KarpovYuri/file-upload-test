<template>
  <div class="file-upload">
    <label
      v-if="label"
      class="file-upload__label"
      :for="inputId"
    >
      {{ label }}
    </label>
    <div
      class="file-upload__wrapper"
      :aria-busy="isLoading && !isCancelling"
      :aria-describedby="
        error ? `${inputId}-error` : hintText ? `${inputId}-hint` : undefined
      "
    >
      <input
        type="file"
        :id="inputId"
        ref="fileInputRef"
        class="file-upload__input"
        :accept="accept"
        :multiple="false"
        @change="handleFileChange"
        aria-hidden="true"
        tabindex="-1"
      />
      <base-button
        @click="onButtonClick"
        :disabled
        role="button"
        :aria-haspopup="false"
        :aria-controls="`${inputId}-status`"
        :aria-describedby="
          error ? `${inputId}-error` : hintText ? `${inputId}-hint` : undefined
        "
      >
        {{ buttonText }}
      </base-button>
      <spinner v-if="isLoading && !isCancelling" />
      <span
        :id="`${inputId}-status`"
        class="file-upload__status-text truncate"
        :class="{ 'file-upload__status-text_uploaded': isUploaded }"
        aria-live="polite"
      >
        {{ fileStatusText }}
      </span>
    </div>
    <span
      v-if="error"
      :id="`${inputId}-error`"
      class="file-upload__error-text"
      role="alert"
    >
      {{ error }}
    </span>
    <p
      v-else-if="hintText"
      :id="`${inputId}-hint`"
      class="file-upload__hint-text"
    >
      {{ hintText }}
    </p>
  </div>
</template>

<script setup lang="ts">
import './FileUpload.css'
import BaseButton from '@/components/UI/BaseButton/BaseButton.vue'
import Spinner from '@/components/UI/Spinner/Spinner.vue'
import { nanoid } from 'nanoid'

import { ref, computed, onUnmounted } from 'vue'

interface FileUploadProps {
  label?: string
  buttonText?: string
  hintText?: string
  accept?: string
  placeholderText?: string
  disabled?: boolean
}

const emit = defineEmits<{
  (e: 'file-uploaded', file: File): void
  (e: 'file-removed'): void
}>()

const props = withDefaults(defineProps<FileUploadProps>(), {
  buttonText: 'Выбрать',
  accept: '',
  placeholderText: 'Файл не выбран',
  disabled: false,
})

const fileInputRef = ref<HTMLInputElement | null>(null)
const selectedFile = ref<File | null>(null)
const isLoading = ref(false)
const isCancelling = ref(false)
const isUploaded = ref(false)
const error = ref<string | null>(null)
const UPLOAD_CANCELLED_MESSAGE = 'Загрузка файла отменена.'
const UPLOAD_DURATION = 1500
let uploadAbortController: AbortController | null = null

const inputId = computed(() => `file-upload-${nanoid()}`)
const fileStatusText = computed(() => {
  if (selectedFile.value) {
    return selectedFile.value.name
  }
  return props.placeholderText
})

const buttonText = computed(() => {
  if (isLoading.value || isCancelling.value) {
    return 'Отменить'
  } else if (isUploaded.value) {
    return 'Удалить'
  } else {
    return props.buttonText
  }
})

const cancelUpload = () => {
  if (uploadAbortController) {
    isCancelling.value = true
    uploadAbortController.abort()
  }
}

const removeFile = () => {
  if (isLoading.value || isCancelling.value) return
  isLoading.value = false
  isCancelling.value = false
  isUploaded.value = false
  error.value = null
  selectedFile.value = null
  if (fileInputRef.value) {
    fileInputRef.value.value = ''
  }
  emit('file-removed')
}

const openFileWindow = () => {
  if (isLoading.value || isCancelling.value || isUploaded.value) return
  if (fileInputRef.value) {
    fileInputRef.value.value = ''
  }
  fileInputRef.value?.click()
}

const onButtonClick = () => {
  if (isLoading.value) {
    cancelUpload()
  } else if (isUploaded.value) {
    removeFile()
  } else {
    openFileWindow()
  }
}

const handleFileChange = async (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files ? target.files[0] : null

  if (!file) return

  error.value = null
  isUploaded.value = false

  selectedFile.value = file
  isLoading.value = true

  uploadAbortController = new AbortController()
  const signal = uploadAbortController.signal

  try {
    // Имитация загрузки и ошибки
    await new Promise((resolve, reject) => {
      const timeoutId = setTimeout(() => {
        if (Math.random() < 0.2) {
          reject(new Error('Error message'))
        } else {
          resolve(true)
        }
      }, UPLOAD_DURATION)

      signal.addEventListener('abort', () => {
        clearTimeout(timeoutId)
        reject(new Error(UPLOAD_CANCELLED_MESSAGE))
      })
    })

    isUploaded.value = true
    if (selectedFile.value) {
      emit('file-uploaded', selectedFile.value)
    }
  } catch (err: any) {
    if (err.message !== UPLOAD_CANCELLED_MESSAGE) {
      error.value = err.message
    }
    selectedFile.value = null
    isUploaded.value = false
  } finally {
    isLoading.value = false
    isCancelling.value = false
    uploadAbortController = null
  }
}

onUnmounted(() => {
  if (uploadAbortController) {
    uploadAbortController.abort()
  }
})
</script>
