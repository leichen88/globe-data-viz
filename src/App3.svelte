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
  import { spring } from "svelte/motion";
  import Glow from "./components/Glow.svelte";
  import Tooltip from "./components/Tooltip.svelte";
  import Legend from "./components/Legend.svelte";
  import data from "./data/data.js";

  // Base spike path (triangle pointing up)
  const spikePath = "M-3,0 L0,-1 L3,0 Z";

  // State
  let countries = topojson.feature(world, world.objects.world_polygons_simplified).features;
  let borders = topojson.feature(boundary, boundary.objects.world_lines_simplified).features;
  let width = $state(600);
  let dragging = $state(false);
  let tooltipData = $state(null);

  // Spring rotations
  let xRotation = spring(0, { stiffness: 0.08, damping: 0.4 });
  let yRotation = spring(-30, { stiffness: 0.17, damping: 0.7 });

  // Derived state
  const height = $derived(width);
  const projection = $derived(geoOrthographic()
    .scale(width / 2)
    .rotate([$xRotation, $yRotation])
    .translate([width / 2, height / 2]));
  const path = $derived(geoPath(projection));

  const heightScale = $derived(scaleLinear()
    .domain([0, max(data, d => d.ref)])
    .range([0, width * 0.3]));

  // Process country data
  countries.forEach(country => {
    const metadata = data?.find(d => d.iso3 === country.properties.iso3);
    country.population = metadata?.ref || 0;
  });

  // Spike calculations
  let spikeData = $derived(
    countries.map(country => {
      const centroid = geoCentroid(country);
      const projected = projection(centroid);
      if (!projected) return null;

      const cx = width / 2;
      const cy = height / 2;
      const dx = projected[0] - cx;
      const dy = projected[1] - cy;
      const scale = width / 2;
      
      const normalX = dx / scale;
      const normalY = dy / scale;
      const spikeLength = heightScale(country.population);

      // Calculate direction angle
      const tipX = projected[0] + normalX * spikeLength;
      const tipY = projected[1] + normalY * spikeLength;
      const angle = (Math.atan2(tipY - projected[1], tipX - projected[0]) + Math.PI/2) * 180/Math.PI;

      return {
        x: projected[0],
        y: projected[1],
        angle,
        length: spikeLength,
        country
      };
    }).filter(Boolean)
  );

  // Auto-rotation
  const degreesPerSecond = 0.25;
  interval(() => {
    if (!dragging && !tooltipData) xRotation.set($xRotation + degreesPerSecond);
  }, 1);

  // Drag interaction
  let globe;
  const DRAG_SENSITIVITY = 3;
  $effect(() => {
    const element = select(globe);
    const dragHandler = drag()
      .on("drag", event => {
        dragging = true;
        xRotation.set($xRotation + event.dx * DRAG_SENSITIVITY);
        yRotation.set($yRotation - event.dy * DRAG_SENSITIVITY);
      })
      .on("end", () => (dragging = false));
    
    element.call(dragHandler);
    return () => element.on(".drag", null);
  });

  // Center on selected country
  $effect(() => {
    if (tooltipData) {
      const center = geoCentroid(tooltipData);
      xRotation.set(-center[0]);
      yRotation.set(-center[1]);
    }
  });
</script>

<div class="chart-container" bind:clientWidth={width}>
  <h1>Global Refugee Displacement Visualization</h1>
  <h2>Population by Country of Origin, 2021</h2>

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
        fill="#333"
        stroke="#1C1C1C"
        stroke-width="0.2"
        on:click={() => (tooltipData = country)}
        on:mouseover={() => (tooltipData = country)}
        on:mouseout={() => (tooltipData = null)}
      />
    {/each}

    {#each borders as border}
      <path d={path(border)} fill="none" stroke="#1C1C1C" stroke-width="0.5" />
    {/each}

    {#each spikeData as spike, index (spike.country.properties.iso3 + '-' + index)}
      <path
        d={spikePath}
        transform={`translate(${spike.x},${spike.y}) rotate(${spike.angle}) scale(1,${spike.length})`}
        fill="red"
        fill-opacity="0.5"
        stroke="red"
        stroke-width="0.5"
        on:mouseover={() => (tooltipData = spike.country)}
        on:mouseout={() => (tooltipData = null)}
      />
    {/each}

    {#if tooltipData}
      {#key tooltipData.properties.iso3}
        <path
          d={path(tooltipData)}
          fill="transparent"
          stroke="white"
          stroke-width="2"
        />
      {/key}
    {/if}
  </svg>

  <Tooltip data={tooltipData} />
  <!-- <Legend data={tooltipData} {heightScale} /> -->
</div>

<style>
  .chart-container {
    position: relative;
    max-width: 600px;
    margin: 0 auto;
  }

  :global(body) {
    background: #282828;
  }

  svg {
    overflow: visible;
  }

  path {
    cursor: pointer;
    transition: stroke-opacity 0.2s;
  }

  path:hover {
    stroke-opacity: 0.8;
  }

  .dragging {
    cursor: move;
  }

  h1, h2 {
    color: white;
    text-align: center;
    margin: 0.5rem 0;
  }

  h1 {
    font-size: 1.75rem;
    font-weight: 600;
  }

  h2 {
    font-size: 1.25rem;
    font-weight: 200;
  }
</style>