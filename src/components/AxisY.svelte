<script>
    import { format } from 'd3-format';
    let { yScale, margin, innerWidth } = $props();
    
    const formatNumber = format("~s");
    let ticks = $derived(yScale.ticks(5));
</script>

<g class="axis y-axis" transform="translate({margin.left}, {margin.top})">
    {#each ticks as tick}
        <line
            x1={0}
            x2={innerWidth}
            y1={yScale(tick)}
            y2={yScale(tick)}
            stroke="white"
            opacity="0.1"
            stroke-width="1"
        />
        <text
            x={-8}
            y={yScale(tick)}
            text-anchor="end"
            fill="white"
            font-size="0.75rem"
            alignment-baseline="middle"
        >
            {formatNumber(tick)}
        </text>
    {/each}
</g>