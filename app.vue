<template>
  <div class="container">
    <!-- <NuxtWelcome /> -->
    <h2>Dodaj plik z danymi</h2>
    <input type="file" @change="handleFileChange" accept=".csv" />
    <!-- <pre>{{ fileContent }}</pre> -->
    <!-- <pre v-html="fileContent"></pre> -->
    <!-- {{fileMatrix[6,1]}} -->
<div class="excel-container">
  <div v-for="(row, rowIndex) in fileMatrix" :key="rowIndex" class="excel-row">
    <!-- Iteruj przez kolumny w danym wierszu -->
    <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="excel-cell">
      {{ cell }}
    </div>
  </div>
</div>


     <!-- Iteruj przez wiersze -->

  </div>
</template>


<script setup>
import { ref } from 'vue';

const fileContent = ref('');
const fileRows = ref([]);
const fileMatrix = ref([]);

const handleFileChange = (event) => {
  const selectedFile = event.target.files[0];
  const reader = new FileReader();

  reader.onload = (e) => {
    fileContent.value = e.target.result;
    fileRows.value = e.target.result.split('\n');
    fileMatrix.value = fileRows.value.map(row => row.split(';')); // Zwróć uwagę na .value
    // console.log(fileContent.value);
    console.log(fileMatrix.value);
  };

  reader.readAsText(selectedFile, 'UTF-8');
};

const modifyCsvData = () => {
  // Tutaj dodaj logikę modyfikacji pliku CSV
  // Pamiętaj, żeby zaktualizować fileContent z wynikiem modyfikacji
};
</script>



<style lang="css">
.container{
  display:flex;
  align-items: center;
  justify-content: center;
  /* background-color: rgb(242, 246, 244); */
}
 .excel-container {
    display: flex;
    flex-direction: column;
    border: 1px solid #ccc;
    padding: 10px;
    margin: 10px;
    width: fit-content;
    overflow-x: auto; /* Dodaje poziomy pasek przewijania, jeśli zawartość jest zbyt szeroka */
  }

  .excel-row {
    display: flex;
    align-items: center;
    border-bottom: 1px solid #ccc; /* Linie oddzielające wiersze */
  }

  .excel-cell {
    padding: 5px;
    width:200px;
    border-right: 1px solid #ccc; /* Linie oddzielające komórki */
    white-space: nowrap; /* Zapobiega zawijaniu tekstu w komórkach */
    overflow: hidden; /* Ukrywa zawartość, która się nie mieści */
    text-overflow: ellipsis; /* Dodaje elipsy na końcu zawartości, która się nie mieści */
  }
</style>
