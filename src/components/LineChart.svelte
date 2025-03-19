<script>
    import { scaleLinear, scaleTime } from 'd3-scale';
    import { Tween } from 'svelte/motion';
    import { cubicInOut } from 'svelte/easing';
    import { interpolateString, interpolate } from 'd3-interpolate';
    import { timeParse } from 'd3-time-format';
    import { line, area, curveMonotoneX } from 'd3-shape';
    import { max, extent } from 'd3-array';
    import data from '../data/LineData.js';
    import AxisX from './AxisX.svelte';
    import AxisY from './AxisY.svelte';

    // Set up dimensions
    let margin = $state({ top: 5, right: 5, bottom: 20, left: 40 });
    let width = $state(0);
    let height = $state(0);

    let innerWidth = $derived(width - margin.left - margin.right);
    let innerHeight = $derived(height - margin.top - margin.bottom);

    const parseTime = timeParse("%Y");

    // Add ranges configuration
    const ranges = [
        
        { value: '5', label: '5 Years' },
        { value: '10', label: '10 Years' },
        { value: '20', label: '20 Years' },
        { value: '30', label: '30 Years' },
        { value: '40', label: '40 Years' },
        { value: '50', label: '50 Years' },
        { value: 'all', label: 'All Years' },
    ];

    // Add selected range state
    let selectedRange = $state('all');

    // Add filteredData derivation
    let filteredData = $derived((() => {
        if (selectedRange === 'all') return data;

        const range = parseInt(selectedRange, 10);
        const parsedYears = data.map(d => parseTime(d.year));
        const maxYear = max(parsedYears);
        const minYear = new Date(maxYear);
        minYear.setFullYear(maxYear.getFullYear() - range);

        return data.filter(d => parseTime(d.year) >= minYear);
    })());

    // Scales
    let xScale = $derived(
        scaleTime()
            .domain(extent(filteredData, d => parseTime(d.year)))
            .range([0, innerWidth])
    );

    let yScale = $derived(
        scaleLinear()
            .domain([0, max(filteredData, d => d.population)])
            .range([innerHeight, 0]).nice()
    );

    // Generators
    let lineGenerator = $derived(line()
        .x(d => xScale(parseTime(d.year)))
        .y(d => yScale(d.population))
        .curve(curveMonotoneX));

    let areaGenerator = $derived(area()
        .x(d => xScale(parseTime(d.year)))
        .y0(yScale(0))
        .y1(d => yScale(d.population))
        .curve(curveMonotoneX));

    // Tween instances with simplified interpolation
    const lineTween = new Tween("", {
        duration: 1500,
        easing: cubicInOut,
        interpolate: interpolate
    });

    const areaTween = new Tween("", {
        duration: 1500,
        easing: cubicInOut,
        interpolate: interpolate
    });

    // Update tweens when data changes
    $effect(() => {
        lineTween.target = lineGenerator(filteredData);
        areaTween.target = areaGenerator(filteredData);
    });


</script>

<div class="top-subsection">
    <h2>Global forcibly displacement trend</h2>
    <div class="filter">
        <!-- <select bind:value={selectedRange}>
            <option value="all">All Years</option>
            <option value="5">Last 5 Years</option>
            <option value="10">Last 10 Years</option>
            <option value="20">Last 20 Years</option>
            <option value="30">Last 30 Years</option>
            <option value="40">Last 40 Years</option>
            <option value="50">Last 50 Years</option>
        </select> -->
        {#each ranges as range}
            <button 
                onclick={() => selectedRange = range.value}
                class:active={selectedRange === range.value}
            >
                {range.label}
            </button>
        {/each}
    </div>
    <div class="chart-container">
        <div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
            <svg {width} {height} aria-label="Trend Analysis Chart" role="img">
                <AxisX {xScale} {height} {margin} />
                <AxisY {yScale} {margin} {innerWidth} />
                <g transform="translate({margin.left},{margin.top})">
                        <!-- Animate the line path -->
                        <path
                            class="path-line"
                            d={lineTween.current}
                            fill="transparent"
                            stroke="#FFC740"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                        <!-- Animate the area path -->
                        <path
                            class="path-area"
                            d={areaTween.current}
                            fill="#FFC740"
                            opacity="0.5"
                        />
                </g>
            </svg>
        </div>
    </div>
</div>

<style>
    .top-subsection {
        background-color: rgba(51, 51, 51, 0.5);
        padding: 16px;
        border-radius: 8px;
        display: flex;
        flex-direction: column;
    }

    .chart-container {
        flex: 1;
        min-height: 200px;
        position: relative;
    }

    .chart {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
    }

    svg {
        overflow: visible;
    }

    .path-line {
        fill: none;
        stroke: #BB9D21;
        stroke-width: 2px;
    }

    .path-area {
        fill: #FAEB00;
        opacity: 0.5;
    }

    h2, p {
        color: white;
        margin: 0;
    }

    h2 {
        font-size: 1.25rem;
        font-weight: 200;
        margin-bottom: 1rem;
    }

    .filter {
        margin-bottom: 1rem;
    }

    select {
        background-color: rgba(102, 102, 102);
        border: 1px solid rgba(102, 102, 102);
        color: white;
        padding: 0.5rem;
        border-radius: 4px;
        font-size: 0.875rem;
    }

    select:focus {
        outline: none;
        border-color: #FAEB00;
    }

    .filter {
        margin-bottom: 1rem;
        display: flex;
        gap: 0.5rem;
        flex-wrap: wrap;
    }

    .filter button {
        background-color: rgba(51, 51, 51);
        border: 1px solid rgba(102, 102, 102);
        color: white;
        padding: 0.25rem 0.25rem;
        border-radius: 2.5px;
        font-size: 0.75rem;
        cursor: pointer;
        transition: all 0.2s ease;
    }

    .filter button:hover {
        background-color: rgba(51, 51, 51, 0.2);
    }

    .filter button.active {
        border-color: #FAEB00;
        background-color: rgba(51, 51, 51, 0.2);
    }

    .filter button:focus {
        outline: none;
        box-shadow: 0 0 0 2px rgba(51, 51, 51, 0.3);
    }

</style>