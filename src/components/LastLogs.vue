<template>
  <div class="p-4">
    <!-- Tabelle anzeigen, wenn Daten verfügbar sind -->
    <div class="w-full overflow-x-auto">
      <table
        class="w-full table-fixed border-collapse border border-gray-800 text-gray-300 text-sm"
      >
        <thead>
          <tr class="bg-gray-800">
            <th class="border border-gray-300 px-4 py-2 text-left w-1/10">
              {{ $t('components.lastLogs.timestamp') }}
            </th>
            <th class="border border-gray-300 px-4 py-2 text-left w-1/10">
              {{ $t('components.lastLogs.level') }}
            </th>
            <th class="border border-gray-300 px-4 py-2 text-left w-2/5">
              {{ $t('components.lastLogs.message') }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(entry, index) in filteredLogs"
            :key="index"
            class="odd:bg-gray-900 even:bg-gray-600"
          >
            <td class="border border-gray-300 px-4 py-2">{{ formatTimestamp(entry.timestamp) }}</td>
            <td
              class="border border-gray-300 px-4 py-2"
              :class="{
                'text-green-600': entry.level === 'INFO',
                'text-red-600': entry.level === 'ERROR',
                'text-yellow-600': entry.level === 'WARNING',
              }"
            >
              {{ entry.level }}
            </td>
            <td class="border border-gray-300 px-4 py-2">{{ entry.message }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { useLogStore } from '@/store/logStore';
const logStore = useLogStore();

// Funktion zum Formatieren des Timestamps (optional)
function formatTimestamp(timestamp) {
  const date = new Date(timestamp);
  return date.toLocaleString();
}

// Computed Property für gefilterte Logs
const filteredLogs = computed(() =>
  logStore.LogsInfo.logs.filter((entry) => !entry.message.includes('EDS_ERR_INVALID_PARAMETER'))
);
</script>

<style scoped></style>
