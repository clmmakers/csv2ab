<script>
  //import { createEventDispatcher } from 'svelte';
  // @ts-ignore
  import { saveAs } from 'file-saver';

  //const dispatch = createEventDispatcher();

  let headers = [];
  let sampleRecords = [];
  let filteredRows = [];
  let totalRecords = 0;
  let lastRecord = [];
  let delimiter = ',';
  let quotedFields = false;
  let lcodcentro = '';
  let ltipolector = '';
  let lidioma = '';
  let mappings = {
    codigocentro: '',
    grupo: '',
    tipolector: '',
    extid: '',
    login: '',
    passwd: '',
    nombre: '',
    apellidos: '',
    dni: '',
    email: '',
    idioma: ''
  };

  const fieldDescriptions = {
    codigocentro: 'OBLIGATORIO: el código del centro. Debe coincidir con el codigo de centro/codigocentro creada en Abies+ y ser el mismo para todos los usuarios a importar. Puede reescribirlo o añadirlo seleccionando "Añadir código de centro manualmente" del selector.',
    grupo: 'OPCIONAL/RECOMENDABLE: nomenclatura de los grupos del centro. Cada usuario tendrá el suyo',
    tipolector: 'OBLIGATORIO: "Alumnado" o "Docentes". Puede añadir el tipo lector seleccionando "Seleccionar manualmente" del selector.',
    extid: 'OBLIGATORIO: identificador único de usuario. también es llamano NIE, UID, UUID, etc. Debe ser único por cada usuario en todo su periplo académico.',
    login: 'OPCIONAL: usuario para acceder a Abies+.',
    passwd: 'OPCIONAL: contraseña para acceder a Abies+.',
    nombre: 'OBLIGATORIO: nombre del usuario.',
    apellidos: 'OBLIGATORIO: apellidos del usuario.',
    dni: 'OPCIONAL: recomendable para "Docentes".',
    email: 'OPCIONAL.',
    idioma: 'OPCIONAL: "es" para castellano, "ca" para catalán, "eu" para euskera.'
  };

  function handleFileUpload(event) {
    const file = event.target.files[0];
    if (file && file.type === 'text/csv') {
      const reader = new FileReader();
      reader.onload = (e) => {
        const text = e.target.result;
        parseCSV(text);
      };
      reader.readAsText(file);
    }
  }

  function detectDelimiterAndQuotes(text) {
    const delimiters = [',', ';', '\t'];
    let detectedDelimiter = ',';
    let detectedQuotedFields = false;

    for (const delimiter of delimiters) {
      const rows = text.split('\n').map(row => row.split(delimiter));
      if (rows[0].length > 1) {
        detectedDelimiter = delimiter;
        detectedQuotedFields = rows[0].some(cell => cell.startsWith('"') && cell.endsWith('"'));
        break;
      }
    }

    return { detectedDelimiter, detectedQuotedFields };
  }

  function parseCSV(text) {
    const { detectedDelimiter, detectedQuotedFields } = detectDelimiterAndQuotes(text);
    delimiter = detectedDelimiter;
    quotedFields = detectedQuotedFields;

    const regex = quotedFields ? new RegExp(`"${delimiter}"`) : new RegExp(delimiter);
    const rows = text.split('\n').map(row => row.split(regex));
    filteredRows = rows.filter(row => row.some(cell => cell.trim() !== ''));
    if (quotedFields) {
      filteredRows = filteredRows.map(row => row.map(cell => cell.replace(/^"|"$/g, '')));
    }

    headers = filteredRows[0];
    sampleRecords = filteredRows.slice(1, 3);
    totalRecords = filteredRows.length;
    lastRecord = filteredRows[filteredRows.length - 1];
    //dispatch('csvData', { headers, sampleRecords });
  }

  function downloadMappedCSV() {
    if (!mappings.codigocentro || (mappings.codigocentro === 'manual' && !lcodcentro) ||
        !mappings.tipolector || (mappings.tipolector === 'manual' && !ltipolector) ||
        !mappings.extid || !mappings.nombre || !mappings.apellidos) {
      alert('No se puede generar el CSV de salida porque faltan campos obligatorios.');
      return;
    }

    if (mappings.codigocentro === 'manual') {
      if (!/^\d{8}$/.test(lcodcentro)) {
        alert('El código de centro debe tener 8 dígitos.');
        return;
      }
    } else {
      const codigocentroIndex = headers.indexOf(mappings.codigocentro);
      if (codigocentroIndex === -1 || !sampleRecords.every(record => /^\d{8}$/.test(record[codigocentroIndex]))) {
        alert('El código de centro debe tener 8 dígitos.');
        return;
      }
    }

    const filteredSampleRecords = filteredRows.filter(record => Object.values(record).some(value => value.trim() !== ''));

    const mappedData = filteredSampleRecords.map(record => {
      return {
        codigocentro: mappings.codigocentro === 'manual' ? lcodcentro : record[headers.indexOf(mappings.codigocentro)],
        grupo: record[headers.indexOf(mappings.grupo)],
        tipolector: mappings.tipolector === 'manual' ? ltipolector : record[headers.indexOf(mappings.tipolector)],
        extid: record[headers.indexOf(mappings.extid)],
        login: record[headers.indexOf(mappings.login)],
        passwd: record[headers.indexOf(mappings.passwd)],
        nombre: record[headers.indexOf(mappings.nombre)],
        apellidos: record[headers.indexOf(mappings.apellidos)],
        dni: record[headers.indexOf(mappings.dni)],
        email: record[headers.indexOf(mappings.email)],
        idioma: mappings.idioma === 'manual' ? lidioma : record[headers.indexOf(mappings.idioma)]
      };
    });

    const csvContent = [
      Object.keys(mappings).join(','),
      ...mappedData.map(row => Object.values(row).join(','))
    ].join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    saveAs(blob, 'mapeo4Ab+.csv');
  }
</script>

<div class="container">
  <input type="file" accept=".csv" on:change={handleFileUpload} class="file-input" />

  {#if headers.length > 0}
  <hr>
  <p>Total de registros: {totalRecords} (se contabiliza la cabecera)</p>
  <table class="csv-preview">
    <thead>
      <tr>
        {#each headers as header}
          <th>{header}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each sampleRecords as record}
        <tr>
          {#each Object.values(record) as value}
            <td>{value}</td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
  <hr>
  <h3>Último registro</h3>
  <table class="csv-preview">
    <thead>
      <tr>
        {#each headers as header}
          <th>{header}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      <tr>
        {#each lastRecord as value}
          <td>{value}</td>
        {/each}
      </tr>
    </tbody>
  </table>
  <hr>
  <h3>Mapeo de columnas</h3>
  {#each Object.keys(mappings) as field}
    <div class="mapping-section">
      <label for={field}>{field}:</label>
      {#if field === 'codigocentro'}
        <select id={field} bind:value={mappings[field]} class="mapping-select">
          <option value="">Seleccione una columna</option>
          {#each headers as header}
            <option value={header}>{header}</option>
          {/each}
          <option value="manual">Añadir código de centro manualmente</option>
        </select>
        {#if mappings[field] === 'manual'}
          <input type="text" placeholder="código de centro" bind:value={lcodcentro} pattern="\d{8}" class="manual-input" />
        {/if}
      {:else if field === 'tipolector'}
        <select id={field} bind:value={mappings[field]} class="mapping-select">
          <option value="">Seleccione una columna</option>
          {#each headers as header}
            <option value={header}>{header}</option>
          {/each}
          <option value="manual">Seleccionar manualmente</option>
        </select>
        {#if mappings[field] === 'manual'}
          <select bind:value={ltipolector} class="manual-select">
            <option value="Alumnado">Alumnado</option>
            <option value="Docentes">Docentes</option>
          </select>
        {/if}
      {:else if field === 'idioma'}
        <select id={field} bind:value={mappings[field]} class="mapping-select">
          <option value="">Seleccione una columna</option>
          {#each headers as header}
            <option value={header}>{header}</option>
          {/each}
          <option value="manual">Seleccionar manualmente</option>
        </select>
        {#if mappings[field] === 'manual'}
          <select bind:value={lidioma} class="manual-select">
            <option value="es">es</option>
            <option value="ca">ca</option>
            <option value="eu">eu</option>
          </select>
        {/if}
      {:else}
        <select id={field} bind:value={mappings[field]} class="mapping-select">
          <option value="">Seleccione una columna</option>
          {#each headers as header}
            <option value={header}>{header}</option>
          {/each}
        </select>
      {/if}
      <p class="field-description {field === 'codigocentro' || field === 'tipolector' || field === 'extid' || field === 'nombre' || field === 'apellidos' ? 'obligatorio' : field === 'grupo' ? 'recomendable' : 'opcional'}">{fieldDescriptions[field]}</p>
    </div>
    <hr>
  {/each}
  <button on:click={downloadMappedCSV} class="download-button">Descargar CSV Mapeado</button>
  {:else}
    <p>No ha seleccionado CSV. Seleccione uno.</p>
  {/if}
</div>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.file-input {
  margin-bottom: 20px;
}

.delimiter-section, .quoted-fields-section, .mapping-section {
  margin-bottom: 20px;
}

.delimiter-section label, .mapping-section label {
  display: inline-block;
  width: 200px;
  margin-right: 10px;
  vertical-align: middle;
}

.delimiter-select, .custom-delimiter-input, .mapping-select, .manual-input, .manual-select {
  display: inline-block;
  padding: 5px;
  width: calc(100% - 220px);
  max-width: 300px;
  vertical-align: middle;
}

.csv-preview {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.csv-preview th, .csv-preview td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.csv-preview th {
  background-color: #f2f2f2;
}

.field-description {
  margin-top: 5px;
  font-size: 0.9em;
}

.field-description.obligatorio {
  color: red;
}

.field-description.recomendable {
  color: orange;
}

.field-description.opcional {
  color: blue;
}

.download-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 1em;
}

.download-button:hover {
  background-color: #0056b3;
}
</style>