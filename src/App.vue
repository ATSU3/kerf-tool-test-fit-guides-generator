<script setup>
import { ref, reactive, watch } from 'vue';

const appState = reactive({
  clicked: false,
  success: false,
  downloadURL: '#',
  displayDownloadText: false,
  errorMessage: '',
  parameters: {
    stw: 14,
    inc: 0.05,
    gap: 20,
    num: 4,
    tbone: 12.7
  }
});

watch(() => appState.parameters.inc, (newValue) => {
  if (newValue >= 0.01) {
    appState.errorMessage = '';
  }
});

const createSVG = async () => {
  appState.clicked = true;
  appState.displayDownloadText = false;

  if (appState.parameters.inc < 0.01) {
    appState.errorMessage = "(pitch) is too small. (pitch) can only be set to 1/100 mm increments. (刻み幅)は1/100mm単位までしか設定できません。";
    appState.success = false;
    return;
  }

  try {
    const response = await fetch(`/kerf_check`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json; charset=utf-8",
      },
      body: JSON.stringify(appState.parameters),
    });

    if (!response.ok) {
      const errorData = await response.json();
      appState.errorMessage = errorData.error || response.statusText;
      appState.success = false;
      return;
    }

    const data = await response.text();
    const blob = new Blob([data], { type: "image/svg+xml" });
    appState.downloadURL = window.URL.createObjectURL(blob);
    appState.success = true;
    appState.displayDownloadText = true;
  } catch (error) {
    appState.errorMessage = "An unexpected error occurred.";
    appState.success = false;
  }
};
</script>

<template>
  <div id="app-1" class="kerf-check-container">
    <h1>Kerf Tool Test Fit Guides Generator</h1>

    <div class="input-container">
      <label>基準幅(start-width): <input v-model.number="appState.parameters.stw" placeholder="14"> mm</label>
      <label>刻み幅(pitch): <input v-model.number="appState.parameters.inc" placeholder="0.05"> mm</label>
      <label>隙間(gap): <input v-model.number="appState.parameters.gap" placeholder="20"> mm</label>
      <label>個数(number-of-kerfs): <input v-model.number="appState.parameters.num" placeholder="4"> 個</label>
      <label>刃径(blade-diameter): <input v-model.number="appState.parameters.tbone" placeholder="12.7"> mm</label>
    </div>

    <button @click="createSVG" class="create-svg-button">Create SVG</button>

    <div v-if="appState.displayDownloadText" class="download-instruction">Click the image to download</div>

    <div v-if="appState.success" class="image-container">
      <a :href="appState.downloadURL" download="kerf_check.svg">
        <img :src="appState.downloadURL" alt="Generated SVG" class="generated-image">
      </a>
    </div>

    <div v-if="appState.errorMessage" class="error-message">
      {{ appState.errorMessage }}
    </div>
  </div>
</template>

<style scoped>
.kerf-check-container {
  text-align: center;
  max-width: 600px;
  margin: auto;
}

.input-container {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.input-container label {
  margin-bottom: 10px;
}

.create-svg-button {
  margin-bottom: 20px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

.download-instruction {
  margin-bottom: 10px;
  font-style: italic;
}

.image-container {
  margin-bottom: 20px;
}

.generated-image {
  cursor: pointer;
  max-width: 100%;
  border: 1px solid #ccc;
}

.error-message {
  color: red;
}
</style>
