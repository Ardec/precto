<template>
  <div class="container">

    <!-- Opcje wczytania dla pliku CSV -->
    <h2>Wczytaj plik CSV</h2>
    <input type="file" @change="handleFileChange" accept=".csv" />
    <br />
    <br />
     <!-- <h2>Wczytaj plik z danymi CSV</h2>
    <input type="file" @change="handleFileChange" accept=".csv, .xlsm" /> -->
    <h2>Wczytaj plik allegro XLSM</h2>
    <input type="file" @change="handleFileChangeAllegro" accept=".xlsm" />
    <br />
    <br />

    

    <button @click="addNewRowsToAllegro">Uzupełnij plik Allegro danymi z CSV</button>

    <div class="excel-container">
      <!-- Wyświetl dane z arkusza Excela (plik allegro) -->
      <div v-for="(row, rowIndex) in worksheet" :key="rowIndex" class="excel-row">
        <div class="excel-cell row-number">{{ rowIndex + 1 }}</div>
        <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="excel-cell">
          {{ cell }}
        </div>
        <br>
        <br>
        <br>
      </div>
         <!-- Wyświetl dane z pierwszego pliku -->
      <div v-for="(row, rowIndex) in fileMatrix" :key="rowIndex" class="excel-row">
        <div class="excel-cell row-number">{{ rowIndex + 1 }}</div>
        <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="excel-cell">
          {{ cell }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import ExcelJS from "exceljs";

const workbook = ref(null); // Przechowuj odniesienie do całego skoroszytu
const worksheet = ref([]); // Przechowuj dane arkusza jako tablicę

const fileMatrix = ref([]);
const fileContent = ref('');
const fileRows = ref([]);


const handleFileChange = (event) => {
  const selectedFile = event.target.files[0];
  const reader = new FileReader();

  reader.onload = (e) => {
    fileContent.value = e.target.result;
    fileRows.value = e.target.result.split('\n');
    fileMatrix.value = fileRows.value.map(row => row.split(';')); 
    console.log(fileMatrix.value);
  };
  reader.readAsText(selectedFile, 'UTF-8');
};


// Odczytaj plik i zaktualizuj dane arkusza
const handleFileChangeAllegro = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();

  reader.onload = (e) => {
    const data = e.target.result;
    workbook.value = new ExcelJS.Workbook();

    // Odczytaj plik XLSM
    workbook.value.xlsx.load(data).then(() => {
      const sheetNumber = 13; // Numer arkusza (indeksowane od 1)
      const loadedWorksheet = workbook.value.getWorksheet(sheetNumber);
      console.log(loadedWorksheet);

      // Czyść i wczytuj dane arkusza
      worksheet.value = [];
      loadedWorksheet.eachRow((row, rowNumber) => {
        const rowData = [];
        row.eachCell({ includeEmpty: true }, (cell) => {
          rowData.push(cell.value); // Użyj `value` zamiast `text`
        });
        worksheet.value.push(rowData);
      });

      console.log(`Arkusz ${loadedWorksheet.name} wczytany do worksheet.`);
    });
  };

  reader.readAsArrayBuffer(file);
};

// Dodaj nowe wiersze i zapisz plik
const addNewRowsToAllegro = async () => {
  if (!workbook.value) {
    alert("Najpierw wczytaj plik XLSM.");
    return;
  }

  const sheetNumber = 13; // Numer arkusza (indeksowane od 1)
  const loadedWorksheet = workbook.value.getWorksheet(sheetNumber);

  // const newRowData = ["Wartość 1", "Wartość 2", "Wartość 3", "", "" , "Wartość 4"];
  // loadedWorksheet.addRow(newRowData);
  // loadedWorksheet.getRow(100).getCell(2).value = "Wartość w wierszu 100 i komórce 2";
    let i = 6; // Zacznij od wiersza 5 w loadedWorksheet
    // Pomiń pierwszy wiersz (nagłówek) rozpoczynając iterację od indeksu 1
    fileMatrix.value.forEach((rowArray, index) => {
      if (index > 2 && i < 200) { // Sprawdź, czy nie jest to nagłówek oraz czy nie przekraczamy 200 wierszy
        loadedWorksheet.addRow([]); // Dodaj pusty wiersz
        loadedWorksheet.getRow(i).getCell(9).value = rowArray[0]; // Numer Katalogowy
        // loadedWorksheet.getRow(i).getCell(9).value = rowArray[1]; // Numer Oryginalny
        loadedWorksheet.getRow(i).getCell(15).value = rowArray[2]; // Nazwa
        loadedWorksheet.getRow(i).getCell(17).value = rowArray[3]; // Opis
        loadedWorksheet.getRow(i).getCell(11).value = rowArray[4]; // Kategoria
        // loadedWorksheet.getRow(i).getCell(11).value = rowArray[5]; // Firma
        // loadedWorksheet.getRow(i).getCell(11).value = rowArray[6]; // Dostępność
        loadedWorksheet.getRow(i).getCell(14).value = rowArray[7]; // Cena netto
        loadedWorksheet.getRow(i).getCell(16).value = rowArray[8]; // Obraz
        loadedWorksheet.getRow(i).getCell(13).value = rowArray[9]; // Ilość w magazynie
        // Dodaj tutaj kolejne instrukcje ustawiające wartości komórek, jeśli to konieczne
        console.log(`Dodano wiersz ${i} z danych CSV do arkusza XLSM.`);
        i += 1; // Zwiększ numer wiersza dla następnej iteracji
      }
    });

  // Zapisz zmiany i wygeneruj nowy plik do pobrania
  try {
    const buffer = await workbook.value.xlsx.writeBuffer();
    const blob = new Blob([buffer], {
      type: "application/vnd.ms-excel.sheet.macroEnabled.12",
    });
    const url = URL.createObjectURL(blob);

    // Tworzenie elementu linku do pobrania pliku
  const downloadLink = document.createElement('a');
  downloadLink.href = url;
  downloadLink.download = "zmodyfikowany_plik.xlsm"; // Ustawienie nazwy pliku z rozszerzeniem .xlsm
  document.body.appendChild(downloadLink);
  downloadLink.click();
  document.body.removeChild(downloadLink);
  URL.revokeObjectURL(url);

    console.log("Plik został zaktualizowany i jest gotowy do pobrania.");
  } catch (error) {
    console.error("Wystąpił błąd podczas zapisywania pliku:", error);
  }
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
  width: 200px;
  border-right: 1px solid #ccc; /* Linie oddzielające komórki */
  white-space: nowrap; /* Zapobiega zawijaniu tekstu w komórkach */
  overflow: hidden; /* Ukrywa zawartość, która się nie mieści */
  text-overflow: ellipsis; /* Dodaje elipsy na końcu zawartości, która się nie mieści */
}

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
