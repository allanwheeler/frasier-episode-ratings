<script>
  import data from '$data/data.js';
  // console.log(data);
  import AxisX from '$components/AxisX.svelte';
  import Tooltip from '$components/Tooltip.svelte';
  import { scaleLinear, scaleBand } from 'd3-scale';
  import { min } from 'd3-array';

  let width = 800;
  let height = 750;

  // Set margins
  const margin = { top: 25, right: 25, bottom: 20, left: 15 };

  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  let seasons = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
  let selSeason = 1;
  let seriesAvg = 8.2;

  // function sortByEpisode() {
  //  // sorted ascending
  //    filteredData.sort((a, b) => a.episode - b.episode);
  // // reassign sorted data to get the function to run
  //  filteredData = filteredData;
  // }

  function sortByEpisode() {
    // sorted ascending
    filteredData = [...filteredData].sort((a, b) => a.episode - b.episode);
  }

  function sortByRating() {
    // sorted descending
    filteredData = [...filteredData].sort(
      (a, b) => b.episodeVsSeriesRating - a.episodeVsSeriesRating
    );
  }

  function filterTopBottom(data) {
    const sortedData = [...data].sort(
      (a, b) => b.averageEpisodeRating - a.averageEpisodeRating
    );
    return [...sortedData.slice(0, 10), ...sortedData.slice(-10)];
  }

  let filteredData = [];

  // Reactive statement will update scales when selSeason binded value changes
  $: if (selSeason === 'top-bottom') {
    filteredData = filterTopBottom(data);
  } else {
    filteredData = data.filter(d => d.season === selSeason);
  }
  $: xScale = scaleLinear()
    .domain([min(data, d => d.averageEpisodeRating), 10])
    .range([0, innerWidth]);

  // Since episodes reactively update by season, the y-axis also needs to be reactive
  $: yScale = scaleBand()
    .domain(filteredData.map(d => d.title))
    .range([0, innerHeight]);

  let hoveredData;
</script>

<main>
  <div class="title-selection">
    <div class="title">
      <p>
        Episodes with
        <span>average ratings >=</span> series average from
      </p>
    </div>
    <div id="selection">
      <select bind:value={selSeason}>
        <option value="top-bottom">Top and bottom 10</option>
        {#each seasons as s}
          <option value={s}>Season {s}</option>
        {/each}
      </select>
    </div>
    {#if selSeason !== 'top-bottom'}
      <div class="buttons">
        <button on:click={sortByRating}>Sort by Rating</button>
        <button on:click={sortByEpisode}>Sort by Episode</button>
      </div>
    {/if}
  </div>

  <div id="chart-container" bind:clientWidth={width} bind:clientHeight={height}>
    <svg {width} {height}>
      <g transform={`translate(${margin.left}, ${margin.top})`}>
        <AxisX {innerHeight} {xScale} {seriesAvg} />
        {#each filteredData as ep}
          <text
            text-anchor={ep.averageEpisodeRating < seriesAvg ? 'start' : 'end'}
            x={ep.averageEpisodeRating < seriesAvg
              ? xScale(seriesAvg) + 8
              : xScale(seriesAvg) - 8}
            y={yScale(ep.title) + yScale.bandwidth() / 2}
            alignment-baseline="middle"
            opacity={hoveredData ? (hoveredData === ep ? 1 : 0.45) : 1}
            fill="#222"
          >
            {#if selSeason === 'top-bottom'}
              S{ep.season}E{ep.episode}: {ep.title}
            {:else}
              E{ep.episode}: {ep.title}
            {/if}
          </text>
          <line
            x1={xScale(seriesAvg)}
            y1={yScale(ep.title) + yScale.bandwidth() / 2}
            x2={xScale(ep.averageEpisodeRating)}
            y2={yScale(ep.title) + yScale.bandwidth() / 2}
            opacity={hoveredData ? (hoveredData === ep ? 1 : 0.45) : 1}
            stroke="#e1e1e1"
            stroke-width="1"
          />
          <circle
            cx={xScale(ep.averageEpisodeRating)}
            cy={yScale(ep.title) + yScale.bandwidth() / 2}
            r={hoveredData === ep ? 6 : 5}
            fill={ep.averageEpisodeRating >= seriesAvg ? '#feb629' : '#e1e1e1'}
            stroke="white"
            opacity={hoveredData ? (hoveredData === ep ? 1 : 0.45) : 1}
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
        {/each}
      </g>
    </svg>
    {#if hoveredData}
      <Tooltip data={hoveredData} {seriesAvg} {xScale} {yScale} />
    {/if}
  </div>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');

  * {
    font-family: inter;
  }

  /* span {
    background-color: rgba(254, 182, 41, 0.3);
    border: 1px solid rgba(254, 182, 41, 1);
    opacity: 1;
    color: #222;
    font-weight: 600;
    padding: 4px 6px;
    border-radius: 3px;
  } */

  span {
    display: inline-block; /* Allows the application of box model properties */
    border-bottom: 3px solid;
    border-color: rgba(254, 182, 41, 1);
    padding-bottom: 2px;
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
