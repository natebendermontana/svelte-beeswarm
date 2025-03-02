<script>
  import data from "$data/data.js";
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { mean, rollups } from "d3-array";
  import Legend from "$components/Legend.svelte";
import {fade} from "svelte/transition";

  let width = 400,
    height = 400;
  const margin = { top: 0, right: 0, left: 0, bottom: 30 };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  const RADIUS = 5;

  // Happiness average by continent
  const continents = rollups(
    data,
    (v) => mean(v, (d) => d.happiness),
    (d) => d.continent,
  ) // Group data by continent and return the group-wide mean
    .sort((a, b) => a[1] - b[1]) // Sort according to value
    .map((d) => d[0]); // Grab the continent name

  // Scales
  $: xScale = scaleLinear()
    .domain([1, 9]) // Alternatively, we could pass .domain(extent(data, d => d.happiness))
    .range([0, innerWidth]);

  let yScale = scaleBand()
    .domain(continents)
    .range([innerHeight, 0])
    .paddingOuter(0.5);

  // Size of circles
  $: radiusScale = scaleSqrt()
    .domain([1, 9]) // Alternatively, we could pass .domain(extent(data, d => d.happiness))
    .range(width < 568 ? [2, 6] : [3, 8]);

  // Color by continent
  const mycolorscale = [
    "#1f77b4",
    "#ff7f0e",
    "#2ca02c",
    "#d62728",
    "#9467bd",
    "#8c564b",
    "#e377c2",
    "#7f7f7f",
    "#bcbd22",
    "#17becf",
  ];
  const colorScale = scaleOrdinal().domain(continents).range(mycolorscale);

  // Simulation
  const simulation = forceSimulation(data);

  $: {
    simulation
      .force(
        "x",
        forceX()
          .x((d) => xScale(d.happiness))
          .strength(0.8),
      )
      .force(
        "y",
        forceY()
          .y((d) => (groupByContinent ? yScale(d.continent) : innerHeight / 2))
          .strength(0.2),
      )
      .force(
        "collide",
        forceCollide().radius((d) => radiusScale(d.happiness) + 1),
      )
      .alpha(0.3) // [0, 1] The rate at which the simulation starts. You should decrease this if you want a slower simulation, or increase it if you want a faster simulation.
      .alphaDecay(0.0005) // [0, 1] The rate at which the simulation alpha approaches 0. you should decrease this if your bubbles are not completing their transitions between simulation states.
      .restart();
  }

  $: nodes = simulation.nodes();
  let nodes = [];
  simulation.on("tick", () => {
    nodes = simulation.nodes();
  });

  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Tooltip from "$components/Tooltip.svelte";
    import { stop_propagation } from "svelte/internal";

  let hovered;
  let hoveredContinent;
  let groupByContinent = false;
</script>

<h1>The Happiest Countries in the World</h1>
<Legend {colorScale} bind:hoveredContinent />
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class="chart-container"
  bind:clientWidth={width}
  on:click={() => {groupByContinent = !groupByContinent;
  hovered = null;}}
  on:mouseleave={() => {
    hovered = null;
  }}
>
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      <AxisX {xScale} height={innerHeight} width={innerWidth} />
      <AxisY {yScale} {groupByContinent} />
      {#if hovered}
        <line
          transition:fade
          x1={hovered.x}
          x2={hovered.x}
          y1={innerHeight}
          y2={hovered.y}
          stroke={colorScale(hovered.continent)}
          stroke-width="2"
        />
      {/if}
      {#each nodes as node, index}
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <circle
        in:fade={{delay: 400 + index * 5}}
          cx={node.x}
          cy={node.y}
          r={radiusScale(node.happiness)}
          fill={colorScale(node.continent)}
          opacity={hovered || hoveredContinent ? hovered === node || hoveredContinent === node.continent 
          ? 1 : 0.3 : 1}
          stroke={hovered || hoveredContinent
            ? hovered === node || hoveredContinent === node.continent
              ? "black"
              : "transparent"
            : "#8d8d8d"}
          stroke-width="1"
          on:mouseover={() => {
            hovered = node;
          }}
          on:focus={() => {
            hovered = node;
          }}
          tabindex="0"
          on:click={(e) => {e.stopPropagation();}}
        />
      {/each}
    </g>
  </svg>
  {#if hovered}
    <Tooltip data={hovered} {colorScale} {width} />
  {/if}
</div>

<style>
  :global(.tick text, .axis-title) {
    font-size: 14px;
    font-weight: 400;
    fill: hsla(212, 10%, 53%, 1);
    user-select: none;
  }

  :global(.axis-title) {
    font-size: 16px;
  }

  h1 {
    font-size: 22px;
    font-weight: 600;
    text-align: center;
    color: rgb(70, 77, 85);
    margin: 0 0 8px 0; /* top, right, bottom, left */
  }

  circle {
    transition:
      stroke 300ms ease,
      opacity 300ms ease;
    cursor: pointer;
  }
</style>
