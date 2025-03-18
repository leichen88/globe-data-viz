<script>
  import world from "./data/world_polygons.json";
  import boundary from "./data/world_lines.json";
  import * as topojson from "topojson-client";
  import { geoOrthographic, geoPath, geoCentroid } from "d3-geo";
  import { max } from "d3-array";
  import { scaleLinear } from "d3-scale";
  import { interval } from "d3-timer";
  import { select } from "d3-selection";
  import { drag } from "d3-drag";
  import { draw } from "svelte/transition";
  import { spring } from "svelte/motion"; // Import the spring function
  import Glow from "./components/Glow.svelte";
  import Tooltip from "./components/Tooltip.svelte";
  import Legend from "./components/Legend.svelte";
  import data from "./data/data.js";

  // State
  let countries = topojson.feature(world, world.objects.world_polygons_simplified).features;
  let borders = topojson.feature(boundary, boundary.objects.world_lines_simplified).features;
  let width = $state(600);
  let dragging = $state(false);
  let tooltipData = $state(null);


// Spike data calculation
let spikeData = $derived(
  countries.map(country => {
    const centroid = geoCentroid(country);
    const projectedPoint = projection(centroid);

    // If the projected point is null, the country is on the back side of the globe
    if (!projectedPoint) return null;

    const cx = width / 2;
    const cy = height / 2;
    const dx = projectedPoint[0] - cx;
    const dy = projectedPoint[1] - cy;
    const scale = width / 2; // Get projection scale

    // Calculate the 3D normal vector
    const normalX = dx / scale;
    const normalY = dy / scale;
    const normalZ = Math.sqrt(1 - normalX * normalX - normalY * normalY); // Ensure the vector is normalized

    // Calculate the spike length based on the population
    const spikeLength = heightScale(country.population || 0);

    // Calculate the end point of the spike using the normal vector
    const x2 = projectedPoint[0] + normalX * spikeLength;
    const y2 = projectedPoint[1] + normalY * spikeLength;

    return {
      x1: projectedPoint[0],
      y1: projectedPoint[1],
      x2: x2,
      y2: y2,
      country: country
    };
  }).filter(d => d !== null)
);

  console.log(borders)
  
  // Spring states for smooth rotations
  let xRotation = spring(0, { stiffness: 0.08, damping: 0.4 }); // Use spring function
  let yRotation = spring(-30, { stiffness: 0.17, damping: 0.7 }); // Use spring function

  // Derived states
  const height = $derived(width);
  const projection = $derived(geoOrthographic()
    .scale(width / 2)
    .rotate([$xRotation, $yRotation]) // Access the spring values with $
    .translate([width / 2, height / 2]));
  const path = $derived(geoPath(projection));
  const colorScale = $derived(scaleLinear()
    .domain([0, max(data, d => d.ref)])
    .range(["#CFFFF2", "#027B68"]));

  const heightScale = $derived(scaleLinear()
    .domain([0, max(data, d => d.ref)])
    .range([0, width * 0.5]));

  // Populate country data
  countries.forEach(country => {
    const metadata = data?.find(d => d.iso3 === country.properties.iso3);
    if (metadata) country.population = metadata.ref;
  });

  console.log(countries)

  // Auto-rotation
  const degreesPerSecond = 0.25;
  interval(() => {
    if (dragging || tooltipData) return;
    xRotation.set($xRotation + degreesPerSecond); // Update spring value
  }, 1);

  // Drag interaction
  let globe;
  const DRAG_SENSITIVITY = 3;
  $effect(() => {
    const element = select(globe);
    const dragHandler = drag()
      .on("drag", event => {
        dragging = true;
        xRotation.set($xRotation + event.dx * DRAG_SENSITIVITY); // Update spring value
        yRotation.set($yRotation - event.dy * DRAG_SENSITIVITY); // Update spring value
      })
      .on("end", () => (dragging = false));

    element.call(dragHandler);
    return () => dragHandler.on("drag", null).on("end", null);
  });

  // Center on selected country
  $effect(() => {
    if (tooltipData) {
      const center = geoCentroid(tooltipData);
      xRotation.set(-center[0]); // Update spring value
      yRotation.set(-center[1]); // Update spring value
    }
  });
</script>

<div class="chart-container" bind:clientWidth={width}>
  <h1>Global refugee displacement at a Glance</h1>
  <h2>Population by country of origin, 2021</h2>
  <svg {width} {height} bind:this={globe} class:dragging>
    <Glow />
    
    <circle
      r={width / 2}
      cx={width / 2}
      cy={height / 2}
      fill="#1c1c1c"
      filter="url(#glow)"
      on:click={() => (tooltipData = null)}
    />

    {#each countries as country}
      <path
        d={path(country)}
        fill="#333333"
        on:click={() => (tooltipData = country)}
        on:focus={() => (tooltipData = country)}
        tabIndex="0"
      />
    {/each}

    {#each borders as border}
<path d={path(border)} fill="none" stroke="#1C1C1C" stroke-width="0.5" />
{/each}

<defs>
  <linearGradient id="spikeGradient" x1="0%" y1="0%" x2="100%" y2="0%">
    <stop offset="0%" style="stop-color:#FFD700;stop-opacity:1" />
    <stop offset="100%" style="stop-color:#FFA500;stop-opacity:1" />
  </linearGradient>
</defs>

{#each spikeData as spike, index (spike.country.properties.iso3 + '-' + index)}
  <line
    x1={spike.x1}
    y1={spike.y1}
    x2={spike.x2}
    y2={spike.y2}
    stroke={colorScale(spike.country.population || 0)}
    stroke-width="3"
    on:click={() => (tooltipData = spike.country)}
    on:mouseover={() => (tooltipData = spike.country)}
    on:mouseout={() => (tooltipData = null)}
    tabIndex="0"
  />
{/each}



    {#if tooltipData}
      {#key tooltipData.properties.iso3}
        <path
          d={path(tooltipData)}
          fill={colorScale(tooltipData.population || 0)}
          stroke="white"
          in:draw
        />
      {/key}
    {/if}
  </svg>

  
  <Tooltip data={tooltipData} />
  <Legend data={tooltipData} {colorScale} />
</div>

<style>
  .chart-container {
    position: relative;
    max-width: 600px;
    margin: 0 auto;
  }

  :global(body) {
    background: rgba(40, 40, 40);
  }

  svg {
    overflow: visible;
  }

  path {
    cursor: pointer;
  }

  .dragging {
    cursor: move;
  }

  h1,
  h2 {
    color: white;
    text-align: center;
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

  /* Typically removing :focus styles is bad accessibility practice,                                                                               but in our case the focused country has its own path outline */
  path:focus,
  path:focus-visible {
    outline: none;
  }
</style>

