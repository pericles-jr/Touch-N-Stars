<template>
  <!-- Nur anzeigen, wenn showModal true ist -->
  <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center">
    <!-- Halbtransparenter Overlay-Hintergrund -->
    <div class="absolute inset-0 bg-black bg-opacity-70" @click="closeModal"></div>
    <div v-if="isLoading">
      <!--Spinner-->
      <div class="flex items-center justify-center w-full h-full">
        <div
          class="w-12 h-12 border-4 border-blue-500 border-t-transparent border-solid rounded-full animate-spin"
        ></div>
      </div>
    </div>
    <!-- Inhalt der Modal -->
    <div v-else class="relative w-full h-full bg-gray-900 z-60 flex items-center justify-center">
      <button
        class="absolute top-4 right-4 text-white hover:text-gray-300 text-2xl font-extrabold z-70"
        @click="closeModal"
        aria-label="Schließen"
      >
        ✕
      </button>
      <div
        ref="imageContainer"
        class="w-full h-full overflow-hidden relative flex items-center justify-center shadow-md shadow-cyan-900"
      >
        <img
          v-if="imageData"
          :src="imageData"
          ref="image"
          @load="onImageLoad"
          class="w-full h-full object-contain cursor-move"
          alt="Vergrößertes Bild"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
/* eslint-disable */
import { ref, watch, nextTick, onBeforeUnmount } from 'vue';
import Panzoom from 'panzoom';

// Props definieren
const props = defineProps({
  showModal: {
    type: Boolean,
    default: false,
  },
  imageData: {
    type: String,
    default: null,
  },
  isLoading: {
    type: Boolean,
    default: true,
  },
});

// Emits definieren
const emits = defineEmits(['close']);

/**
 * Schließt die Modal, indem ein "close"-Event emittiert wird
 */
function closeModal() {
  emits('close');
}

// Refs für Panzoom
const image = ref(null);
let panzoomInstance = null;

/**
 * Initialisiert Panzoom auf dem Bild
 */
const initializePanzoom = () => {
  if (image.value) {
    panzoomInstance = Panzoom(image.value, {
      maxZoom: 40,
      minZoom: 0.5,
      bounds: true,
      boundsPadding: 0.5,
    });
  }
};

/**
 * Bereinigt die Panzoom-Instanz
 */
const destroyPanzoom = () => {
  if (panzoomInstance) {
    panzoomInstance.dispose();
    panzoomInstance = null;
  }
};

// Handler für das Laden des Bildes
const onImageLoad = () => {
  // Warte, bis das Bild vollständig geladen ist
  nextTick(() => {
    destroyPanzoom(); // Vorherige Instanz bereinigen
    initializePanzoom(); // Neue Instanz initialisieren
  });
};

// Beobachte Änderungen an showModal, um Panzoom zu initialisieren oder zu bereinigen
watch(
  () => props.showModal,
  (newVal) => {
    if (newVal) {
      // Wenn das Modal geöffnet wird, Panzoom initialisieren nach dem Laden des Bildes
      // Der @load-Handler im img-Tag kümmert sich darum
    } else {
      // Wenn das Modal geschlossen wird, Panzoom bereinigen
      destroyPanzoom();
    }
  }
);

// Bereinige Panzoom-Instanz, wenn die Komponente zerstört wird
onBeforeUnmount(() => {
  destroyPanzoom();
});
</script>

<style scoped>
/* Sicherstellen, dass der Modal-Container die volle Fläche nutzt */
.modal-container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

/* Bild-Container */
.image-container {
  overflow: hidden;
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Bild selbst */
.image-container img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  cursor: move;
}

/* Schließen-Button spezifische Stile */
button[aria-label='Schließen'] {
  z-index: 70;
}
</style>
