<template>
  <div class="page-container">
    <div class="container">
      <textarea
        v-model="articleText"
        placeholder="Collez votre texte ici..."
        rows="10"
        :disabled="isLoading"
      ></textarea>

      <div class="controls">
        <select v-model="selectedTone" :disabled="isLoading">
          <option value="" disabled selected>Sélectionnez un ton</option>
          <option value="factuel">Factuel / Objectif</option>
          <option value="professionnel">Professionnel</option>
          <option value="amusant">Amusant / Sarcastique</option>
        </select>

        <button
          @click="generateSummary"
          :disabled="isLoading || !articleText || !selectedTone"
        >
          {{ isLoading ? "Génération en cours..." : "Générer le Résumé" }}
        </button>
      </div>

      <div class="result">
        <div id="summary-output">
          {{
            summary ||
            (isLoading ? "Analyse en cours..." : "... le résumé apparaîtra ici")
          }}
        </div>
      </div>

      <p v-if="error" class="error-message">
        {{ error }}
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const articleText = ref("");
const selectedTone = ref("");
const summary = ref(null);
const isLoading = ref(false);
const error = ref(null);

const API_URL = "http://localhost:8000/api/v1/summarize";

const generateSummary = async () => {
  if (!articleText.value || !selectedTone.value) return;

  isLoading.value = true;
  summary.value = null;
  error.value = null;

  try {
    const response = await fetch(API_URL, {
      method: "POST",
      body: JSON.stringify({
        article_text: articleText.value,
        tone: selectedTone.value,
      }),
      headers: {
        "Content-Type": "application/json",
      },
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.detail || `Erreur lors de la requête: ${response.status}`);
    }

    summary.value = data.summary;
  } catch (err) {
    error.value = `Échec de l'opération : ${err.message}. Vérifiez que votre backend est lancé sur localhost:8000.`;
    summary.value = "Erreur de génération. Veuillez vérifier les logs.";
  } finally {
    isLoading.value = false;
  }
};
</script>

<style>
:root {
  --black: #040222;
  --grey: #3b3b3b;
  --white: #fafaff;
  --flash: #d81e5b;
  --green: #1b998b;
  --shine: #f4e04d;
}
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

body {
  overflow: hidden;
}
.page-container {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: red;
}
.container {
  font-family: Arial, sans-serif;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  width: 70vw;
  max-width: 1000px;
  height: 80vh;
  margin: auto;
  padding: 20px;
  background: var(--black);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
}
textarea {
  width: 100%;
  padding: 15px;
  border: 4px dashed var(--green);
  border-radius: 6px;
  box-sizing: border-box;
  resize: vertical;
  font-size: 20px;
  color: var(--white);
}
textarea::placeholder {
  color: var(--shine);
  opacity: 1;
}
.controls {
  display: flex;
  justify-content: space-around;
  gap: 20px;
  margin-bottom: 30px;
  align-items: center;
}
select,
button {
  width: 33%;
  padding: 12px 8px;
  border-radius: 6px;
  font-size: 16px;
}
button {
  background-color: var(--green);
  color: var(--white);
  border: none;
  cursor: pointer;
  transition: background-color 0.2s;
}
button:hover:not(:disabled) {
  background-color: var(--white);
  color: var(--green);
}
button:disabled {
  background-color: var(--white);
  color: var(--green);
  opacity: 0.3;
  cursor: not-allowed;
  box-shadow: 1px 1px 3px var(--black);
}
#summary-output {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  border: 4px solid var(--green);
  height: 200px;
  overflow-y: auto;
  max-width: 850px;
  margin: auto;
  white-space: pre-wrap;
  background-color: var(--black);
  border-radius: 8px;
  color: var(--green);
}
.error-message {
  color: var(--flash);
  font-weight: bold;
  margin-top: 15px;
}
@media (max-width: 768px) {
  textarea {
    font-size: 16px;
  }
  .controls {
    flex-direction: column;
  }
  select,
  button {
    font-size: 12px;
  }

  #summary-output {
    font-size: 16px;
  }
}
</style>
