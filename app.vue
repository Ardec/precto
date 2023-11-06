<template>
  <div class="container">
    <h2>Wczytaj plik z danymi CSV</h2>
    <input type="file" @change="handleFileChange" accept=".csv, .xlsm" />
    <h2>Wczytaj plik allegro XLSM</h2>
    <input type="file" @change="handleFileChangeAllegro" accept=".csv, .xlsm" />
    <br>
    <br>
    <button>Uzupełnij plik Allegro</button>
    <br>
    <br>
    <button>Pobierz plik Allegro</button>

    <div class="excel-container">
      <!-- Wyświetl dane z pierwszego pliku -->
      <div v-for="(row, rowIndex) in fileMatrix" :key="rowIndex" class="excel-row">
        <div class="excel-cell row-number">{{ rowIndex + 1 }}</div>
        <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="excel-cell">
          {{ cell }}
        </div>
      </div>

      <!-- Wyświetl dane z arkusza Excela (plik allegro) -->
      <div v-for="(row, rowIndex) in worksheet" :key="rowIndex" class="excel-row">
        <div class="excel-cell row-number">{{ rowIndex + 1 }}</div>
        <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="excel-cell">
          {{ cell }}
        </div>
      </div>
    </div>
  </div>
</template>



<script setup>
import { ref } from 'vue';
import ExcelJS from 'exceljs';

const fileContent = ref('');
const fileContentAllegro = ref('');
const fileRows = ref([]);
const fileRowsAllegro = ref([]);
const fileMatrix = ref([]);
const fileMatrixAllegro = ref([]);
const worksheet = ref([]); // Dodaj zmienną dla arkusza

const handleFileChange = (event) => {
  const selectedFile = event.target.files[0];
  const reader = new FileReader();

  reader.onload = (e) => {
    fileContent.value = e.target.result;
    fileRows.value = e.target.result.split('\n');
    fileMatrix.value = fileRows.value.map(row => row.split(';')); // Zwróć uwagę na .value
    console.log(fileMatrix.value);
  };
  reader.readAsText(selectedFile, 'UTF-8');
};

const handleFileChangeAllegro = (event) => {
  const selectedFileAllegro = event.target.files[0];
  const readerAllegro = new FileReader();

  readerAllegro.onload = (e) => {
    const data = e.target.result;
    const workbook = new ExcelJS.Workbook();

    // Odczytaj plik XLSM
    workbook.xlsx.load(data).then(function () {
      const sheetNumber = 13; // Numer trzeciego arkusza (indeksowane od 0)
      const loadedWorksheet = workbook.getWorksheet(sheetNumber);
      // console.log("workBOOK",workbook)

      worksheet.value = []; // Wyczyść istniejącą zawartość arkusza
      loadedWorksheet.eachRow((row, rowNumber) => {
        const rowData = [];
        row.eachCell({ includeEmpty: true }, (cell) => {
          rowData.push(cell.text);
        });
        worksheet.value.push(rowData);
      });
      console.log(`Arkusz ${loadedWorksheet.name} wczytany do worksheet.`);
    });
  };

  readerAllegro.readAsArrayBuffer(selectedFileAllegro);
};
</script>



<style lang="css">
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

<style>
  .excel-container {
    display: flex;
    flex-direction: column;
    border: 1px solid #ccc;
    padding: 10px;
    margin: 10px;
    width: fit-content;
    overflow-x: auto;
  }

  .excel-row {
    display: flex;
    align-items: center;
    border-bottom: 1px solid #ccc;
  }

  .excel-cell {
    padding: 5px;
    border-right: 1px solid #ccc;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .row-number {
    font-weight: bold;
    padding-right: 10px; /* Dodaje odstęp między numeracją a zawartością komórki */
  }
</style>

