<script>
    import { draw } from 'svelte/transition';

    import { extent } from 'd3-array';
    import { scaleLinear } from 'd3-scale';
    import { line, curveBasis } from 'd3-shape';

    // props
    export let data2;
    export let max;
    export let show;

    // scales
    const xScale = scaleLinear()
        .domain(extent(data2.map(d => d.age)))
        .range([5, max]);

    const yScale = scaleLinear()
        .domain(extent(data2.map(d => d.temp)))
        .range([1, max]);

    // the path generator
    const pathLine = line()
        .x(d => xScale(d.age))
        .y(d => yScale(d.temp))
        .curve(curveBasis);
</script>

<svg viewBox="0 0 200 200">

        <path transition:draw={{duration: 2000}}
              d={pathLine(data2)} />

</svg>

<style>
    path {
        stroke: purple;
        stroke-width: 2;
        fill: none;
        stroke-linecap: round;
    }

</style>