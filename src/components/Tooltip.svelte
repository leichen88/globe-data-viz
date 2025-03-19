<script>
  let { data, total, selectedYear, pointSum } = $props();
  import { fade } from "svelte/transition";
  import { format } from "d3-format";
  const suffixFormat = d => format(".2~s")(d).replace(/G/, "B");
</script>

{#if data?.properties.gis_name}
  <div transition:fade>
    <h2>{data.properties.gis_name}</h2>
    <h3>
      People registered: {pointSum ? suffixFormat(pointSum) : 'N/A'} in {selectedYear}
    </h3>
  </div>
{:else}
  <div transition:fade>
    <h2>People registered worldwide</h2>
    <h3>{suffixFormat(total)} in {selectedYear}</h3>
  </div>
{/if}

<style>
  div {
    position: absolute;
    bottom: 40px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 1000;
    text-align: center;
    padding: 1rem 2rem;
    min-width: 220px;
  }

  h2 {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: 0.5rem;
    color: white;
  }

  h3 {
    font-size: 1.15rem;
    font-weight: 100;
    color: rgba(255, 255, 255, 0.7);
    margin: 0;
  }
</style>