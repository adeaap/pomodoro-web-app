<template>
  <div v-if="isVisible" class="dialog-backdrop" @click.self="closeDialog">
    <div class="dialog-content">
      <h1 class="dialog-title">Settings</h1>
      <slot></slot>
      <button class="close-button" @click="closeDialog">OK</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  isVisible: Boolean
})

const emits = defineEmits(['update:isVisible'])

const closeDialog = () => {
  emits('update:isVisible', false)
}
</script>

<style scoped>
.dialog-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.6);
}

.dialog-title {
  font-size: 1.5em;
  margin-bottom: 20px;
  color: #d7e0ff;
  margin-bottom: 30px;
}

.dialog-content {
  /* background-color: white; */
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  border-radius: 8px;
  width: 30%;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.close-button {
  font-weight: 600;
  width: 90%;
  margin-top: 60px;
  padding: 10px 20px;
  background-color: #f87060;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
}
</style>
