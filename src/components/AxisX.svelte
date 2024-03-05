<script>
  export let innerHeight;
  export let xScale;
  export let seriesAvg;

  let ticks = [6, 7, 8.2, 9, 10];
</script>

{#each ticks as tick, i}
  {#if tick === seriesAvg}
    <line
      x1={xScale(seriesAvg)}
      x2={xScale(seriesAvg)}
      y1={0}
      y2={innerHeight}
      stroke="#b7b7b7"
      stroke-width="1"
      stroke-dasharray="3, 3"
    />
    <text
      x={xScale(seriesAvg)}
      y={innerHeight}
      dy="0.9rem"
      text-anchor="middle"
    >
      Avg {seriesAvg}
    </text>
    <text x={xScale(seriesAvg)} y={0} dy="-0.5rem" text-anchor="middle">
      Avg {seriesAvg}
    </text>
  {:else}
    <line
      x1={xScale(tick)}
      x2={xScale(tick)}
      y1={0}
      y2={innerHeight}
      stroke="#b7b7b7"
      stroke-width="0.5"
    />
    <text x={xScale(tick)} y={innerHeight} dy="0.9rem" text-anchor="middle">
      {tick} stars
    </text>
    <text x={xScale(tick)} y={0} dy="-0.9rem" text-anchor="middle">
      {tick} stars
    </text>
  {/if}
{/each}
