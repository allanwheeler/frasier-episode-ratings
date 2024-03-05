<script>
  export let data;
  export let seriesAvg;
  export let xScale;
  export let yScale;

  $: x = xScale(data.averageEpisodeRating);
  $: y = yScale(data.title);

  import { fly } from 'svelte/transition';
</script>

<div
  in:fly
  class="tooltip"
  style="left: {data.averageEpisodeRating >= seriesAvg ? x + 20 : x - 15}px; 
				 top: {y + 35}px"
>
  {data.averageEpisodeRating}
</div>

<style>
  .tooltip {
    position: absolute;
    pointer-events: none;
    transition:
      top 250ms ease,
      left 250ms ease;
  }
</style>
