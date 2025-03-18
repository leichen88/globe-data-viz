<script>
  export let colorScale;
  export let data;

  $: percentOfMax = (data?.population / colorScale.domain()[1]) * 100;

  import { fade } from "svelte/transition";

  import { format } from "d3-format";
  const suffixFormat = d => format(".2~s")(d).replace(/G/, "B");
</script>

<div class="legend" aria-hidden="true">
  <span class="label">0</span>
  <div
    class="bar"
    style:background="linear-gradient(to right, {colorScale.range()[0]}, {colorScale.range()[1]})"
  >
    {#if percentOfMax}
      <span transition:fade class="line" style="left: {percentOfMax}%;"></span>
    {/if}
  </div>
  <span class="label">{suffixFormat(colorScale.domain()[1])}</span>
</div>

<style>
  .legend {
    display: flex;
    gap: 6px;
    position: absolute; /* add absolute positioning */
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
  }

  .bar {
    height: 15px;
    width: 300px; /* fixed width for better control */
    position: relative;
  }

  .label {
    color: white;
    font-size: 0.85rem;
    user-select: none;
    white-space: nowrap; /* prevent text wrapping */
  }

  .line {
    position: absolute;  /* Changed from relative */
    top: 0;
    height: 100%;       /* Full height of container */
    transform: translateX(-50%); /* Center the line */
  }
</style>

