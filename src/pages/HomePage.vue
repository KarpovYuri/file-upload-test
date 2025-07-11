<template>
  <main>
    <file-upload
      label="Label"
      hint-text="Hint text"
      @file-uploaded="handleFileUploaded"
      @file-removed="handleFileRemoved"
    />
  </main>
</template>

<script setup lang="ts">
import FileUpload from '@/components/commons/FileUpload/FileUpload.vue'

const handleFileUploaded = (file: File) => {
  console.log('Файл успешно загружен:', file.name, file)
}

const handleFileRemoved = () => {
  console.log('Файл удален.')
}
</script>
