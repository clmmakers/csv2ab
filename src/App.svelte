<script>
  // @ts-ignore
  import CsvUploader from "./components/CsvUploader.svelte";
  import image from "./assets/paracsvAdapterAB.png";
  import logointef from "./assets/logointef.png";
  import logoab from "./assets/logoabies.png";

  let csvData = [];
  let headers = [];
  let sampleRecords = [];
  let mappedColumns = {};

  // function handleCsvUpload(data) {
  //   csvData = data.records;
  //   headers = data.headers;
  //   sampleRecords = csvData.slice(0, 5); // Display first 5 records as sample
  // }

  // function handleColumnMapping(mapping) {
  //   mappedColumns = mapping;
  // }
</script>

<main>
  <div class="header">
    <img src={logointef} alt="logo INTEF" style="height: 40px;">
    <img src={logoab} alt="logo Abies" style="height: 40px;">
</div>
  <div class="info">
    <h2>
      Información sobre los campos del .csv para la importación masiva de
      Usuarios
    </h2>
    <img src={image} alt="info sobre los campos del csv" />
    <h4>
      Recuerde que en cada carga de importación de usuarios por .csv, Abies+:
    </h4>
    <ul>
      <li>
        Comprobará si cada usuario (linea del csv) existe en Abies+,
        IDENTIFICANDO A CADA USUARIO POR SU EXTID:
      </li>
      <ul>
        <li>
          Si el usuario existe, se actualizarán los campos que se hayan
          modificado en el csv
        </li>
        <li>
          Si el usuario no existe, se creará un nuevo usuario con los campos del
          csv
        </li>
      </ul>
      <li>
        Los usuarios que existen en Abies+ y que no vienen en el csv son
        borrados, SALVO QUE TENGAN PRESTAMOS PENDIENTES
      </li>
    </ul>
  </div>
  <hr />
  <h1>Adecuador CSV para importación masiva Abies+</h1>
  <!-- <CsvUploader on:csvUpload={handleCsvUpload} /> -->
  <CsvUploader />
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
  }
  .info {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    margin: 0;
    text-align: center;
  }
  .info img {
    max-width: 100%;
    height: auto;
  }
  .info ul {
    text-align: left;
    padding-left: 20px;
  }
  .info li {
    text-align: left;
  }
  .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 50px; /* Espacio en la parte superior/inferior y a los lados */
            background-color: white; /* Color de fondo para el header */
            position: sticky; /* Hace que el header sea sticky */
            top: 0; /* Se adhiere a la parte superior de la ventana */
            z-index: 1000; /* Asegura que el header esté por encima de otros elementos */
        }
        .header img {
            max-width: 100%; /* Asegura que las imágenes no excedan el ancho del contenedor */
            height: auto;    /* Mantiene la proporción de las imágenes */
        }
</style>
