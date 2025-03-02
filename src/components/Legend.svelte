<script>
    export let colorScale;
    export let hoveredContinent;

    const ticks = colorScale.domain();
</script>

<div class="legend" on:mouseleave={() => (hoveredContinent = null)}>
    {#each ticks as continent}
        <p
            on:mouseover={() => {
                hoveredContinent = continent;
            }}
            class:unhovered={hoveredContinent && hoveredContinent !== continent}
        >
            <span style="background-color: {colorScale(continent)}" />
            {continent}
        </p>
    {/each}
</div>

<style>
    .legend {
        display: flex;
        flex-direction: row;
        justify-content: center;
        flex-wrap: wrap;
        column-gap: 10px;
        row-gap: 5px;
        margin-bottom: 5px;
    }

    span {
        display: inline-block;
        width: 9px;
        height: 9px;
        border-radius: 50%;
        border: 1px solid #8d8d8d;
    }

    p {
        font-size: 12px;
        text-transform: uppercase;
        display: flex;
        column-gap: 3px;
        align-items: center;
        transition: opacity 300ms ease;
        cursor: pointer;
    }

    .unhovered {
        opacity: 0.3;
    }
</style>
