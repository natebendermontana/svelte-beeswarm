<script>
    export let data;
    export let colorScale;
    export let width;
    import { fly, fade } from "svelte/transition";

    let TooltipWidth;

    const xNudge = 5;
    const yNudge = 5;
    // if the tooltip is going to overflow the right side of the screen,
    // we move it to the left of the cursor
    $: xPosition =
        data.x + TooltipWidth + xNudge > width
            ? data.x - TooltipWidth - xNudge
            : data.x + xNudge;

    $: yPosition = data.y + yNudge;
</script>

<div
    class="tooltip"
    in:fly={{ y: 10, duration: 200, delay: 200 }}
    out:fade
    style="position:absolute; top: {yPosition}px; left: {xPosition}px;"
    bind:clientWidth={TooltipWidth}
>
    <h1>
        {data.country}
    </h1>
    <div class="info">
        <span class="score">
            {data.happiness.toFixed(1)}/10
        </span>
        <span
            class="continent"
            style="background: {colorScale(data.continent)}"
        >
            {data.continent}
        </span>
    </div>
    <span class="bar background" />
    <span
        class="bar foreground"
        style="background: {colorScale(data.continent)};
    width:{data.happiness * 10}%;"
    />
</div>

<style>
    .tooltip {
        background-color: white;
        padding: 8px 6px;
        border-radius: 3px;
        box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
        pointer-events: none;
        transition:
            top 300ms ease,
            left 300ms ease;
    }

    .info {
        display: flex;
        justify-content: space-between;
        column-gap: 8px;
        align-items: center;
    }

    .score {
        font-size: 12px;
    }

    .continent {
        font-size: 12px;
        padding: 3px;
        color: white;
        text-transform: uppercase;
        border-radius: 3px;
    }

    h1 {
        font-size: 18px;
        font-weight: 500;
        margin: 0 0 4px 0;
    }

    .bar {
        position: absolute;
        bottom: 0;
        left: 0;
        height: 4px;
        width: 100%;
    }

    .bar.background {
        background: #eee;
    }
</style>
