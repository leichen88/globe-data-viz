<script>
    import { timeFormat } from 'd3-time-format';
    let { xScale, height, margin } = $props();
    
    const formatYear = timeFormat("%Y");
    let ticks = $derived(xScale.ticks(5));
</script>

<g class="axis x-axis" transform="translate({margin.left}, {height - margin.bottom})">
    {#each ticks as tick}
        <!-- Tick line -->
        <line
            x1={xScale(tick)}
            x2={xScale(tick)}
            y1={0}
            y2={4}
            stroke="white"
            opacity="5"
        />
        
        <!-- Tick label -->
        <text
            x={xScale(tick)}
            y={18}
            text-anchor="middle"
            fill="white"
            font-size="0.75rem"
        >
            {formatYear(tick)}
        </text>
    {/each}
    
    <!-- Base line -->
    <path
        d={`M0,0H${xScale.range()[1]}`}
        stroke="white"
        opacity="1"
        stroke-width="1"
    />
</g>