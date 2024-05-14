<script>
  import Graph from './Graph.svelte';
  let yearRangeMin = 2000;
  let yearRangeMax = 2023;
  let populationMin = 0;
  let selectedCategory = 'biofuel';
  let dataType = 'consumption';

  function updateCategoryLabel(category, type) {
      return `${category.replace('_', ' ')} ${type.charAt(0).toUpperCase() + type.slice(1)}`.toUpperCase();
  }

  function updateVisualization() {
      if (yearRangeMin > yearRangeMax) {
          const temp = yearRangeMin;
          yearRangeMin = yearRangeMax;
          yearRangeMax = temp;
      }
      
  }
</script>

<style>
  .container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 20px;
  }

  #controls {
      margin-bottom: 20px;
      display: flex;
      justify-content: space-between;
  }

  .control-group {
      display: flex;
      align-items: center;
  }

  .slider-group, .select-group {
      margin-right: 20px;
  }

  .slider-label, .select-label {
      margin-bottom: 5px;
      display: block;
  }

  .slider-group input[type="range"] {
      width: 100%;
      margin-bottom: 5px;
  }

  .slider-container {
      display: flex;
      flex-direction: column;
      width: 200px;
  }

  select {
      border: 1px solid #d9d9d9; 
      background-color: #444;
      color: #d9d9d9;
      padding: 5px;
      font-family: 'Roboto', sans-serif;
  }

  label, .slider-label {
      color: #000; 
  }

  input[type="range"] {
      accent-color: #808080; 
  }

  #visualization {
      margin-top: 20px;
  }
</style>

<div class="container">
  <div id="controls">
      <div class="control-group">
          <div class="slider-group">
              <label for="yearRangeMin" class="slider-label">Year Range: {yearRangeMin} - {yearRangeMax}</label>
              <div class="slider-container">
                  <input type="range" id="yearRangeMin" name="yearRangeMin" min="2000" max="2023" bind:value={yearRangeMin} on:input={updateVisualization}>
                  <input type="range" id="yearRangeMax" name="yearRangeMax" min="2000" max="2023" bind:value={yearRangeMax} on:input={updateVisualization}>
              </div>
          </div>
          <div class="slider-group">
              <label for="populationMin" class="slider-label">Minimum Population: {populationMin}</label>
              <input type="range" id="populationMin" name="populationMin" min="0" max="1000000000" step="1000000" bind:value={populationMin} on:input={updateVisualization}>
          </div>
      </div>
      <div class="control-group">
          <div class="select-group">
              <label for="energyCategory" class="select-label">Energy Category:</label>
              <select id="energyCategory" name="energyCategory" bind:value={selectedCategory} on:change={() => { updateCategoryLabel(selectedCategory, dataType); updateVisualization(); }}>
                  <option value="biofuel">Biofuel</option>
                  <option value="coal">Coal</option>
                  <option value="electricity">Electricity</option>
                  <option value="fossil_fuel">Fossil Fuel</option>
                  <option value="gas">Gas</option>
                  <option value="hydro">Hydro</option>
                  <option value="low_carbon">Low Carbon</option>
                  <option value="nuclear">Nuclear</option>
                  <option value="oil">Oil</option>
                  <option value="other_renewable">Other Renewable</option>
                  <option value="renewables">Renewables</option>
                  <option value="solar">Solar</option>
                  <option value="wind">Wind</option>
              </select>
          </div>
          <div class="select-group">
              <label for="dataType" class="select-label">Data Type:</label>
              <select id="dataType" name="dataType" bind:value={dataType} on:change={() => { updateCategoryLabel(selectedCategory, dataType); updateVisualization(); }}>
                  <option value="consumption">Consumption</option>
                  <option value="production">Production</option>
              </select>
          </div>
      </div>
  </div>
  <div id="visualization">
      <Graph {yearRangeMin} {yearRangeMax} {populationMin} {selectedCategory} {dataType} {updateCategoryLabel} />
  </div>
</div>
