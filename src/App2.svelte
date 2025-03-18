<script>
  import GlobeViz from './components/GlobeViz.svelte';
  import Line from './components/LineChart.svelte';
  import KeyStats from './components/KeyStats.svelte';
</script>

<div class="page-container">
  <GlobeViz />
  <div class="right-section">
    <Line />
    <KeyStats />
  </div>
</div>

<style>
  .page-container {
    display: grid;
    grid-template-columns: 3fr 2fr;
    height: 100vh;
    overflow-y: auto;
  }

  .right-section {
    display: grid;
    grid-template-rows: 2fr 3fr;
    gap: 16px;
    background-color: #222222;
    padding: 16px;
  }

  @media (max-width: 768px) {
    .page-container {
      grid-template-columns: 1fr;
    }

    .right-section {
      grid-template-rows: auto auto;
    }
  }
</style>