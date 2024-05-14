<script>
  // Write your JS here, or import other files
</script>

<main>
  <body>
      <div class="controls">
          <label for="year">Year:</label>
          <select id="year"></select>
          
          <label for="category">Energy Category:</label>
          <select id="category">
              <option value="biofuel_consumption">Biofuel Consumption</option>
              <option value="coal_consumption">Coal Consumption</option>
              <option value="electricity_generation">Electricity Generation</option>
              <option value="fossil_fuel_consumption">Fossil Fuel Consumption</option>
              <option value="gas_consumption">Gas Consumption</option>
              <option value="hydro_consumption">Hydro Consumption</option>
              <option value="low_carbon_consumption">Low Carbon Consumption</option>
              <option value="nuclear_consumption">Nuclear Consumption</option>
              <option value="oil_consumption">Oil Consumption</option>
              <option value="other_renewable_consumption">Other Renewable Consumption</option>
              <option value="renewables_consumption">Renewables Consumption</option>
              <option value="solar_consumption">Solar Consumption</option>
              <option value="wind_consumption">Wind Consumption</option>
          </select>
          
          <label for="population">Population Size:</label>
          <input type="range" id="population" min="0" max="1000000000" step="10000000">
      </div>
      
      <div id="chart" class="chart"></div>
  
      <script src="https://d3js.org/d3.v7.min.js"></script>
      <script src="script.js"></script>
  </body>
</main>

<style>
  /* Write your CSS here */
</style>
