<script>
  export let data;
  export let margin;
  export let seriesAvg;
  export let xScale;
  export let yScale;
  export let sortBy;
  export let seasonRank;

  const xOffset = 12;
  const yOffset = -6;

  $: x =
    xScale(data.averageEpisodeRating) +
    margin.left +
    (data.averageEpisodeRating >= seriesAvg ? xOffset : -xOffset - 40);

  // Adjust y based on sortBy values
  $: y =
    sortBy === 'episode'
      ? yScale(data.episode - 1) + margin.top + yOffset
      : yScale(seasonRank[data.episode - 1]) + margin.top + yOffset;

  import { fly } from 'svelte/transition';
</script>

<div
  in:fly={{ duration: 900 }}
  class="tooltip"
  style="left: {x}px; 
				 top: {y}px; text-align: {data.averageEpisodeRating > seriesAvg
    ? 'left'
    : 'right'}"
>
  {data.averageEpisodeRating} / 10
</div>

<style>
  .tooltip {
    position: absolute;
    font-size: 0.75rem;
    pointer-events: none;
  }
</style>
