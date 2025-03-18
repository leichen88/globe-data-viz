<script>
  import Globe from 'globe.gl';
  import Tooltip from './Tooltip1.svelte';
  import Legend from './Legend1.svelte';
  import data from '../data/data.js';
  import data2 from '../data/data2.js';
  import world from '../data/world_polygons.json';
  import boundary from '../data/world_lines.json';
  import * as topojson from 'topojson-client';
  import { scaleLinear } from 'd3-scale';
  import { max } from 'd3-array';
  import { format } from 'd3-format';
  import earthTexture from '../assets/earth-night.jpg';

  let width = $state(0); 
  let height = $state(0);

  const formatter = d => format('.2~s')(d).replace(/G/, 'B');

  const uniqueYears = [...new Set(data2.map(d => d.year))].sort((a, b) => a - b);

  // Set default year to 2023 if it exists in uniqueYears, otherwise fall back to the first available year
  let selectedYear = $state(uniqueYears.includes(2023) ? 2023 : uniqueYears[0]);

  let globeInstance;
  let hoveredCountry = $state();
  let countries = topojson.feature(world, world.objects.world_polygons_simplified).features;

  // Populate country data
  countries.forEach(country => {
    const metadata = data?.find(d => d.iso3 === country.properties.iso3);
    if (metadata) {
      country.population = metadata.ref;
      country.lat = metadata.lat;
      country.lon = metadata.lon;
    }
  });

  // Reactive statement to filter data based on selectedYear
  let filteredData = $derived(data2.filter(d => d.year === selectedYear));
  // Add derived total population
  let totalPopulation = $derived(filteredData.reduce((acc, d) => acc + d.population, 0));

  let countryPointSum = $derived(
    hoveredCountry
      ? filteredData
          .filter(d => d.iso3 === hoveredCountry.properties.iso3)
          .reduce((sum, d) => sum + d.population, 0)
      : null
  );

  // Define color and size scale
  let colorScale = $derived(
    scaleLinear()
      .domain([0, max(filteredData, d => d.population)])
      .range(['#FAEB00', '#EF4A60'])
  );
  let sizeScale = $derived(
    scaleLinear()
      .domain([0, max(filteredData, d => d.population)])
      .range([0, 0.6])
  );

  let gData = $derived(
    filteredData.map(d => ({
      lat: +d.lat,
      lng: +d.lon,
      size: sizeScale(d.population),
      color: colorScale(d.population),
    }))
  );

  $effect(() => {
    const container = document.getElementById('globeViz');
    globeInstance = new Globe(container)
      .pointAltitude('size')
      .pointColor('color');

    globeInstance
      .globeImageUrl(earthTexture)
      .backgroundColor('#222222')
      .polygonsData(countries)
      .polygonAltitude(0.01)
      .polygonCapColor(() => 'rgba(51, 51, 51, 0.5)')
      .polygonSideColor(() => 'rgba(51, 51, 51, 0.1)')
      .polygonStrokeColor(() => '#000000')
      .onPolygonHover(hover => {
        hoveredCountry = hover;
        globeInstance
          .polygonAltitude(d => (d === hover ? 0.02 : 0.001))
          .polygonSideColor(d => (d === hover ? '#00000080' : 'transparent'))
          .polygonCapColor(d => (d === hover ? '#ffffff' : 'transparent'));
      })
      .polygonsTransitionDuration(300);

    // Enable auto-rotation
    globeInstance.controls().autoRotate = true;
    globeInstance.controls().autoRotateSpeed = 0.5;
    globeInstance.controls().enableZoom = false;

    // Handle resize events
    const handleResize = () => {
      if (!container) return;
      globeInstance.width(container.clientWidth).height(container.clientHeight);
    };
    handleResize();
    window.addEventListener('resize', handleResize);

    // Cleanup function
    return () => {
      window.removeEventListener('resize', handleResize);
      if (globeInstance) {
        globeInstance.scene().children.forEach(child => {
          if (child.dispose) child.dispose();
        });
        container.innerHTML = '';
      }
    };
  });

  $effect(() => {
    // Update points data when gData changes
    if (globeInstance) {
      globeInstance.pointsData(gData);
    }
  });
</script>

<div class="globe-container">
  <div class="overlay-text">
    <h1>Global forcibly displaced people at a Glance</h1>
    <h2>Population by country of origin, 2001-2023</h2>
  </div>
  <select class="year-selector" bind:value={selectedYear}>
    {#each uniqueYears as year}
      <option value={year}>{year}</option>
    {/each}
  </select>
  <div id="globeViz"></div>
  <Tooltip data={hoveredCountry} total={totalPopulation} selectedYear={selectedYear} pointSum={countryPointSum} />
  <Legend data={countries} {colorScale} />
</div>

<style>
  .globe-container {
    position: relative;
    width: 100%;
    height: 100%;
    min-height: 600px;
    overflow: hidden;
    background-color: #222222;
  }

  .year-selector {
    position: absolute;
    background-color: rgba(255, 255, 255, 0.1);
    border: 1px solid #666;
    color: white;
    padding: 0.5rem;
    border-radius: 4px;
    font-size: 0.875rem;
    top: 20px;
    left: 20px;
    z-index: 10;
  }
  .year-selector:focus {
        outline: none;
        border-color: #BB9D21;
    }

  #globeViz {
    width: 100%;
    height: 100%;
    touch-action: none;
  }

  .overlay-text {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    z-index: 10;
    pointer-events: none;
  }

  h1,
  h2 {
    color: white;
    margin: 0;
  }

  h1 {
    font-size: 1.75rem;
    font-weight: 600;
    margin-bottom: 0.35rem;
  }

  h2 {
    font-size: 1.25rem;
    font-weight: 200;
    margin-bottom: 1rem;
  }
</style>
  