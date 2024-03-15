<script>
  import data from '$data/data.js';
  // console.log(data);
  import AxisX from '$components/AxisX.svelte';
  import Tooltip from '$components/Tooltip.svelte';
  import { scaleLinear } from 'd3-scale';
  import { rank, min, ascending } from 'd3-array';
  import { flip } from 'svelte/animate';
  import { fade } from 'svelte/transition';

  let width = 800;
  let height = 800;

  const margin = { top: 25, right: 25, bottom: 20, left: 15 };

  $: innerWidth = width - margin.right - margin.left;
  $: innerHeight = height - margin.top - margin.bottom;

  let seasons = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
  let selectSeason = 1;
  let seriesAvg = 8.2;
  let sortBy = 'episode';
  let hoveredData;

  $: filteredData = data.filter(d => d.season === selectSeason);

  $: seasonRank = rank(
    filteredData.map(d => d.averageEpisodeRating),
    (a, b) => ascending(b, a) || 1
  );

  $: xScale = scaleLinear()
    .domain([min(data, d => d.averageEpisodeRating), 10])
    .range([0, innerWidth]);

  $: yScale = scaleLinear().domain([0, 24]).range([0, innerHeight]);
</script>

<main>
  <div class="title-selection">
    <div class="title">
      <p>Average user ratings for Frasier for</p>
    </div>
    <div id="selection">
      <select bind:value={selectSeason}>
        {#each seasons as s}
          <option value={s}> Season {s}</option>
        {/each}
      </select>
    </div>
    <div class="buttons">
      <button on:click={() => (sortBy = 'rating')}>Sort by rating</button>
      <button on:click={() => (sortBy = 'episode')}>Sort by episode</button>
    </div>
  </div>
  <div id="chart-container" bind:clientWidth={width} bind:clientHeight={height}>
    <svg {width} {height}>
      <g transform={`translate(${margin.left}, ${margin.top})`}>
        <AxisX {innerHeight} {xScale} {seriesAvg} />
        {#each data.filter(d => d.season === selectSeason) as ep, i (ep.title)}
          <g
            animate:flip={{ delay: 25, duration: 1000 }}
            in:fade
            out:fade
            transform="translate(0,{yScale(
              sortBy === 'episode' ? ep.episode - 1 : seasonRank[i]
            )})"
          >
            <text
              class="ep-title"
              text-anchor={ep.averageEpisodeRating < seriesAvg
                ? 'start'
                : 'end'}
              x={ep.averageEpisodeRating < seriesAvg
                ? xScale(seriesAvg) + 10
                : xScale(seriesAvg) - 2}
              dx={-5}
              y={0}
              alignment-baseline="middle"
              fill="#222"
              on:mouseover={() => {
                hoveredData = ep;
              }}
              on:mouseleave={() => {
                hoveredData = null;
              }}
              on:focus={() => {
                hoveredData = ep;
              }}
            >
              E{ep.episode}: {ep.title}
            </text>
            <line
              x1={xScale(seriesAvg)}
              x2={xScale(ep.averageEpisodeRating)}
              y1={0}
              y2={0}
              stroke="#e1e1e1"
              stroke-width="1"
            />
            <circle
              cx={xScale(ep.averageEpisodeRating)}
              cy={0}
              r={6}
              fill={ep.averageEpisodeRating >= seriesAvg
                ? '#feb629'
                : '#d5d5d5'}
              stroke="white"
              on:mouseover={() => {
                hoveredData = ep;
              }}
              on:mouseleave={() => {
                hoveredData = null;
              }}
              on:focus={() => {
                hoveredData = ep;
              }}
            />
          </g>
        {/each}
      </g>
    </svg>
    {#if hoveredData}
      <Tooltip
        data={hoveredData}
        {xScale}
        {yScale}
        {margin}
        {sortBy}
        {seasonRank}
        {seriesAvg}
      />
    {/if}
  </div>
  <p class="source">Source: IMDb</p>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');

  * {
    font-family: inter;
  }

  select {
    border: none;
    font-size: 0.875rem;
  }

  .title-selection {
    font-size: 0.875rem;
    user-select: none;
    display: flex;
    margin: 0.5rem 0.7rem 0rem 0.75rem;
    gap: 0.5rem;
    align-items: baseline;
  }

  .buttons {
    display: flex;
    margin: 0rem 0.75rem 1rem 0.75rem;
  }

  button {
    padding: 4px 6px;
    background: #fff;
    border: 1px solid #919191;
    color: #222; /* White text color */
    font-size: 0.8rem;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      transform 0.2s ease; /* Smooth transition for hover and click effects */
  }

  button:focus {
    background-color: #ededed; /* Slightly darker grey on hover */
  }

  .buttons button:first-child {
    border-top-left-radius: 5px;
    border-bottom-left-radius: 5px;
    border-right: none;
  }

  .buttons button:last-child {
    border-top-right-radius: 5px;
    border-bottom-right-radius: 3px;
  }

  .buttons button:not(:last-child) {
    margin-right: -1px;
  }

  #chart-container {
    font-size: 0.7rem;
    position: relative;
    height: 90vh;
  }
</style>
