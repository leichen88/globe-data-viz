<script>
    import data from '../data/data.js';
    import { format } from 'd3-format';
    let width = $state(0); 
    let height = $state(0);
  
    // Sort data by date and get latest 100 records
    const formattedData = data
      .map(d => ({
        ...d,
        population: Number(d.population),
        date: new Date(d.year) // Ensure date is parsed correctly
      }))
      .sort((a, b) => b.year - a.year) // Sort descending by date
      .slice(0, 100); // Get first 100 items
  
    const formatter = format('.2~s');
  </script>
  
  <div class="bottom-subsection">
      <h2>Latest key statistics</h2>
      <div class="chart-container">
          <div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
            <div class="key-stats">
                <div class="stats-grid">
                  <div class="header-row">
                    <span>Population</span>
                    <span>Admin 1</span>
                    <span>Country</span>
                    <span>Year</span>
                  </div>
                  
                  {#each formattedData as item, i (item.adm1_pcode)}
                    <div class="data-row">
                      <span class="population">
                          {formatter(item.population).replace(/G/, 'B')}
                      </span>
                      <span class="admin1">{item.adm1_name}</span>
                      <span class="country">{item.coa}</span>
                      <span class="year">{item.year}</span>
                      
                    </div>
                  {/each}
                </div>
              </div>
          </div>
      </div>
  </div>
  
  <style>
    /* .bottom-subsection {
      background-color: rgba(51, 51, 51, 0.5);
      padding: 16px;
      border-radius: 8px;
    }
  
    .chart-container {
      flex: 1;
      min-height: 450px;
      position: relative;
    }
  
    .chart {
      position: absolute;
      width: 100%;
      height: 100%;
    } */
    .bottom-subsection {
    background-color: rgba(51, 51, 51, 0.5);
    padding: 16px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
  }

  .chart-container {
    flex: 1; /* Take remaining space */
    min-height: 0; /* Allow container to shrink */
    position: relative;
  }

  .chart {
    height: 100%;
    width: 100%;
    position: absolute;
  }

  h2,
  p {
    color: white;
    margin: 0;
  }

  h2 {
    font-size: 1.25rem;
    font-weight: 200;
    margin-bottom: 1rem;
  }

  .key-stats {
    height: 100%;
    display: flex;
    flex-direction: column;
  }

  .stats-grid {
    flex: 1;
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* 4 columns */
    gap: 0.75rem 1rem;
    overflow-y: auto;
  }

  .header-row {
    grid-column: 1 / -1;
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* 4 columns */
    gap: 0.75rem 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid #333;
    font-weight: 500;
    color: #888;
  }

  .header-row span:last-child {
    justify-self: end; /* Align last column to the right */
  }

  .data-row {
    grid-column: 1 / -1;
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* 4 columns */
    gap: 0.75rem 1rem;
    padding: 0.3rem 0;
    border-bottom: 1px solid #333;
    color: #ffffff;
    font-size: 0.9rem;
  }

  .data-row span:last-child {
    justify-self: end; /* Align last column to the right */
  }

  .rank {
    color: #FAEB00;
    font-weight: 500;
  }

  .year {
    font-weight: 500;
  }

  .country {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .population {
    color: #FAEB00;
    font-weight: 700;
    text-align: left;
    font-feature-settings: 'tnum';
    font-variant-numeric: tabular-nums;
  }

  @media (max-width: 768px) {
    .key-stats {
      padding: 1rem;
    }

    .chart-container {
      flex: 1;
      min-height: 200px;
      position: relative;
    }

    h2 {
      font-size: 1.1rem;
    }

    .stats-grid {
      font-size: 0.85rem;
      grid-template-columns: repeat(2, 1fr); /* 2 columns on smaller screens */
    }

    .header-row,
    .data-row {
      grid-template-columns: repeat(2, 1fr); /* 2 columns on smaller screens */
    }

    .header-row span:nth-child(3),
    .header-row span:nth-child(4),
    .data-row span:nth-child(3),
    .data-row span:nth-child(4) {
      display: none; /* Hide less important columns on smaller screens */
    }

    .header-row span:last-child,
    .data-row span:last-child {
      justify-self: start; /* Reset alignment for smaller screens */
    }
  }
  </style>