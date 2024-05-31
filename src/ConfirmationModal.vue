<template>
    <div class="fixed inset-0 flex items-center justify-center bg-gray-800 bg-opacity-75">
      <div class="bg-white rounded-lg p-6">
        <h2 class="text-xl font-bold mb-4">Confirm Delete</h2>
        <p class="mb-6">Are you sure you want to delete this todo?</p>
        <div class="flex justify-end">
          <button @click="cancel" class="bg-gray-500 text-white rounded px-4 py-2 mr-2">Cancel</button>
          <button @click="confirm" class="bg-red-500 text-white rounded px-4 py-2">Delete</button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: ['show', 'onConfirm', 'onCancel'],
    methods: {
      confirm() {
        this.onConfirm();
      },
      cancel() {
        this.onCancel();
      }
    }
  };
  </script>
  
  <style scoped>
  /* Optional styles */
  </style>
  