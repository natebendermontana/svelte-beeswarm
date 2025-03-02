<script>
  export let xScale;
  export let width;
  export let height; 

$: ticks = xScale.ticks(4); 

</script>

<g class="axis x">
  {#each ticks as tick}
    <g class="tick" transform="translate({xScale(tick)}, 0)">
      <text x="4" y={height} >{tick}</text>
      <line x1="0" y1="0" x2="0" y2={height} stroke="#c8c8c8"></line>
    </g>
  {/each}
  <text class="axis-title" x={width} y={height}
  text-anchor="end"
  dominant-baseline="hanging"
  dy="8"
  >
    Happiness, out of 10 &rarr;
  </text>
  </g>

