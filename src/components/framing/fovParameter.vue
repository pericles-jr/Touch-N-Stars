<template>
  <div
    class="flex flex-row items-center justify-between w-full border border-gray-300 p-2 rounded-xl transition-all duration-200 hover:border-cyan-500 focus-within:border-cyan-500 hover:shadow-lg"
  >
    <div>
      <p class="text-sm font-medium text-gray-500">
        {{ $t('components.framing.fovSettings.fov') }}
      </p>
    </div>
    <input
      class="text-black px-4 h-10 w-28 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-cyan-500 focus:border-cyan-500 transition-all duration-200"
      type="number"
      v-model="framingStore.fov"
      min="0"
      max="10000"
    />
  </div>
</template>
<script setup>
import { useFramingStore } from '@/store/framingStore';

const framingStore = useFramingStore();
</script>
