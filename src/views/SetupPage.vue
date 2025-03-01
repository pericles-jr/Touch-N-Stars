<template>
  <div class="min-h-screen bg-gray-900 flex items-center justify-center p-4">
    <div class="w-full max-w-md mx-4">
      <!-- Progress Indicator -->
      <div class="mb-8">
        <div class="flex justify-center space-x-2">
          <div
            v-for="step in totalSteps"
            :key="step"
            class="h-2 w-2 rounded-full transition-all duration-300"
            :class="{
              'bg-cyan-500 scale-125': currentStep === step,
              'bg-gray-600': currentStep !== step,
            }"
          ></div>
        </div>
      </div>
      <!-- Setup Steps -->
      <transition
        name="slide-fade"
        mode="out-in"
        @before-enter="beforeEnter"
        @after-enter="afterEnter"
      >
        <div
          v-if="currentStep === 1"
          key="welcome"
          class="bg-gray-800 p-6 md:p-8 rounded-lg shadow-lg transform transition-all duration-500"
          :class="{
            'scale-95 opacity-0': !isVisible,
            'scale-100 opacity-100': isVisible,
          }"
        >
          <h1 class="text-3xl font-bold text-white mb-4">{{ t('setup.welcome') }}</h1>
          <p class="text-gray-300 mb-6">{{ t('setup.description') }}</p>
          <button
            @click="currentStep++"
            class="w-full bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-3 px-4 rounded-lg transition-all transform hover:scale-105"
          >
            {{ t('common.confirm') }}
          </button>
        </div>

        <div
          v-else-if="currentStep === 2"
          key="language"
          class="bg-gray-800 p-8 rounded-lg shadow-lg"
        >
          <h2 class="text-2xl font-bold text-white mb-6">{{ t('setup.selectLanguage') }}</h2>
          <select
            v-model="selectedLanguage"
            class="w-full px-4 py-3 bg-gray-700 text-gray-300 rounded-lg mb-6 focus:ring-2 focus:ring-cyan-500 focus:border-cyan-500"
          >
            <option
              v-for="lang in availableLanguages"
              :key="lang.code"
              :value="lang.code"
              class="bg-gray-800"
            >
              {{ lang.name }}
            </option>
          </select>
          <div class="flex justify-between">
            <button
              @click="currentStep--"
              class="bg-gray-600 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded-lg"
            >
              {{ t('common.cancel') }}
            </button>
            <button
              @click="saveLanguage"
              class="bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-2 px-4 rounded-lg"
            >
              {{ t('common.confirm') }}
            </button>
          </div>
        </div>

        <div
          v-else-if="currentStep === 3 && isTauri"
          key="gps"
          class="bg-gray-800 p-8 rounded-lg shadow-lg"
        >
          <h2 class="text-2xl font-bold text-white mb-6">{{ t('setup.gpsConfiguration') }}</h2>
          <div class="space-y-4">
            <div class="flex items-center gap-2">
              <div class="flex-1">
                <label class="block text-sm font-medium text-gray-400 mb-1">Latitude</label>
                <input
                  v-model="latitude"
                  type="text"
                  class="w-full px-3 py-2 bg-gray-700 text-gray-300 rounded-md"
                />
              </div>
              <div class="flex-1">
                <label class="block text-sm font-medium text-gray-400 mb-1">Longitude</label>
                <input
                  v-model="longitude"
                  type="text"
                  class="w-full px-3 py-2 bg-gray-700 text-gray-300 rounded-md"
                />
              </div>
            </div>
            <button
              @click="getCurrentLocation"
              class="w-full bg-gray-600 hover:bg-gray-500 text-white py-2 px-4 rounded-md"
            >
              {{ t('components.settings.coordinates') }}
            </button>
            <div v-if="gpsError" class="text-red-400 text-sm">{{ gpsError }}</div>
          </div>
          <div class="flex justify-between mt-6">
            <button
              @click="currentStep--"
              class="bg-gray-600 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded-lg"
            >
              {{ t('common.cancel') }}
            </button>
            <button
              @click="saveGPS"
              class="bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-2 px-4 rounded-lg"
            >
              {{ t('common.confirm') }}
            </button>
          </div>
        </div>

        <div
          v-else-if="currentStep === 4 && isTauri"
          key="instance"
          class="bg-gray-800 p-8 rounded-lg shadow-lg"
        >
          <h2 class="text-2xl font-bold text-white mb-6">
            {{ t('setup.instanceConfiguration') }}
          </h2>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-1">Instance Name</label>
              <input
                v-model="instanceName"
                type="text"
                class="w-full px-3 py-2 bg-gray-700 text-gray-300 rounded-md"
                placeholder="My Instance"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-1">IP Address / FQDN</label>
              <input
                v-model="instanceIP"
                type="text"
                class="w-full px-3 py-2 bg-gray-700 text-gray-300 rounded-md"
                placeholder="192.168.x.x"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-400 mb-1">Port</label>
              <input
                v-model="instancePort"
                type="text"
                class="w-full px-3 py-2 bg-gray-700 text-gray-300 rounded-md"
                placeholder="5000"
              />
            </div>
            <div class="flex justify-between mt-6">
              <button
                @click="currentStep--"
                class="bg-gray-600 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded-lg"
              >
                {{ t('common.cancel') }}
              </button>
              <button
                @click="saveInstance"
                class="bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-2 px-4 rounded-lg"
              >
                {{ t('common.confirm') }}
              </button>
            </div>
          </div>
        </div>

        <div v-else key="complete" class="bg-gray-800 p-8 rounded-lg shadow-lg">
          <h2 class="text-2xl font-bold text-white mb-6">{{ t('setup.completeSetup') }}</h2>
          <button
            @click="completeSetup"
            class="w-full bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-3 px-4 rounded-lg"
          >
            {{ t('common.confirm') }}
          </button>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useI18n } from 'vue-i18n';
import { getAvailableLanguages } from '@/i18n';
import { useRouter } from 'vue-router';
import { useSettingsStore } from '@/store/settingsStore';

const { locale, t } = useI18n();
const router = useRouter();
const settingsStore = useSettingsStore();

const currentStep = ref(1);
const totalSteps = computed(() => 5);
const isVisible = ref(true);

function beforeEnter() {
  isVisible.value = false;
}

function afterEnter() {
  isVisible.value = true;
}

const selectedLanguage = ref(locale.value);
const latitude = ref('');
const longitude = ref('');
const gpsError = ref(null);
const isTauri = ref(window.__TAURI__ !== undefined);
const instanceName = ref('');
const instanceIP = ref('');
const instancePort = ref('');

const availableLanguages = getAvailableLanguages();

function saveLanguage() {
  locale.value = selectedLanguage.value;
  settingsStore.setLanguage(selectedLanguage.value);
  currentStep.value++;
}

async function getCurrentLocation() {
  if (!isTauri.value) {
    gpsError.value = 'GPS only available in Tauri builds';
    return;
  }

  try {
    const { checkPermissions, requestPermissions, getCurrentPosition } = await import(
      '@tauri-apps/plugin-geolocation'
    );

    let permissions = await checkPermissions();
    if (permissions.location === 'prompt' || permissions.location === 'prompt-with-rationale') {
      permissions = await requestPermissions(['location']);
    }

    if (permissions.location === 'granted') {
      const pos = await getCurrentPosition({
        enableHighAccuracy: true,
        timeout: 10000,
        maximumAge: 0,
      });

      latitude.value = pos.coords.latitude.toFixed(6);
      longitude.value = pos.coords.longitude.toFixed(6);
      gpsError.value = null;
    } else {
      gpsError.value = 'Location permission not granted';
    }
  } catch (error) {
    gpsError.value = error.message || 'Failed to get GPS location';
  }
}

function saveGPS() {
  settingsStore.setCoordinates({
    latitude: latitude.value,
    longitude: longitude.value,
  });
  currentStep.value++;
}

function saveInstance() {
  // Validate Nina connection details
  if (!instanceName.value.trim()) {
    alert(t('components.settings.errors.instanceNameRequired'));
    return;
  }

  if (!instanceIP.value) {
    alert(t('components.settings.errors.invalidIPFormat'));
    return;
  }

  // Validate port number
  const port = parseInt(instancePort.value);
  if (isNaN(port) || port < 1 || port > 65535) {
    alert(t('components.settings.errors.invalidPortRange'));
    return;
  }

  settingsStore.addInstance({
    name: instanceName.value,
    ip: instanceIP.value,
    port: instancePort.value,
  });
  currentStep.value++;
}

function completeSetup() {
  settingsStore.completeSetup();
  localStorage.setItem('setupCompleted', 'true');
  router.push('/');
}
</script>

<style scoped>
.slide-fade-enter-active {
  transition: all 0.5s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.3s ease-in;
}

.slide-fade-enter-from {
  opacity: 0;
  transform: translateX(30px);
}

.slide-fade-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

@media (max-width: 640px) {
  .slide-fade-enter-from {
    transform: translateX(15px);
  }
  .slide-fade-leave-to {
    transform: translateX(-15px);
  }
}
</style>
