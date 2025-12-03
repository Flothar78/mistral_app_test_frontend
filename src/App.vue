<template>
  <div class="title">
    Résumé et Traduction 
    <div>Avec Mistral 7B</div>
  </div>
  <div class="container">
    <textarea
      v-model="articleText"
      placeholder="Collez votre article ici..."
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
      <h2>Résumé Obtenu</h2>
      <div id="summary-output">
        {{ summary || (isLoading ? "Analyse en cours..." : "Le résumé apparaîtra ici.") }}
      </div>
    </div>

    <p v-if="error" class="error-message">
      {{ error }}
    </p>
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

<style scoped>
.container {
  /* border: 2px solid red; */
  width: 50vw;
  height: 70vh;
  margin: 0 auto;
  padding: 20px;
  background-color: white;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
}
.title {
  text-align: center;
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 20px;
  color: #007bff;
}
textarea {
  width: 100%;
  padding: 15px;
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 6px;
  box-sizing: border-box;
  resize: vertical;
  font-size: 16px;
}
.controls {
  display: flex;
  gap: 20px;
  margin-bottom: 30px;
  align-items: center;
}
select,
button {
  padding: 12px 8px;
  border-radius: 6px;
  font-size: 16px;
}
select {
  flex-grow: 1;
  border: 1px solid #007bff;
}
button {
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s;
}
button:hover:not(:disabled) {
  background-color: #0056b3;
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

#summary-output {
  white-space: pre-wrap;
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  min-height: 50px;
  border: 1px solid #eee;
  /* Style pour le texte d'attente */
  color: #666;
  font-style: italic;
}
.error-message {
  color: #dc3545;
  font-weight: bold;
  margin-top: 15px;
}
@media (max-width: 768px) {
  .controls {
    flex-direction: column;
  }
  select,
  button {
    font-size: 14px;
  }
}
</style>
