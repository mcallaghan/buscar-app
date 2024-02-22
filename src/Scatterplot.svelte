<script>
	import { onMount } from 'svelte';
	import { scaleLinear, scaleLog } from 'd3-scale';
  import incrspace from '@stdlib/array/incrspace';
  import roundn from '@stdlib/math-base-special-roundn';
	export let points;

	let svg;
	let width = 500;
	let height = 500;

	const padding = { top: 20, right: 40, bottom: 40, left: 40 };
  $: xmin = Math.floor({points}["points"][0].x * 20) / 20
  $: xTicks = incrspace(xmin,1.01,0.05).map((x) => roundn(x,-2));


	$: xScale = scaleLinear()
		.domain([xmin, 1])
		.range([padding.left, width - padding.right]);

	$: yScale = scaleLinear()
		.domain([0, 1])
		.range([height - padding.bottom, padding.top]);

	//$: xTicks = width > 180 ? [xmin, 1] : [0, 1];

	$: yTicks = height > 180 ? [0, 0.05, 0.1, 0.33, 0.5, 1] : [0, 0.05, 0.1, 0.33, 0.5, 1];

	onMount(resize);

	function resize() {
		({ width, height } = svg.getBoundingClientRect());
	}
</script>

<svelte:window on:resize={resize} />

<svg bind:this={svg}>
	<!-- y axis -->
	<g class="axis y-axis">
		{#each yTicks as tick}
			<g class="tick tick-{tick}" transform="translate(0, {yScale(tick)})">
				<line x1={padding.left} x2={xScale(22)} />
				<text x={padding.left - 8} y="+4">{tick}</text>
			</g>
		{/each}
    <g class="axlabel" transform="translate(0, {yScale(0.9)}) rotate(-90)">
      <text x={padding.left + 4} y="+12">p value</text>
    </g>
	</g>

	<!-- x axis -->
	<g class="axis x-axis">
		{#each xTicks as tick}
			<g class="tick" transform="translate({xScale(tick)},0)">
				<line y1={yScale(0)} y2={yScale(13)} />
				<text y={height - padding.bottom + 16}>{tick}</text>
			</g>
		{/each}
    <g class="axlabel" transform="translate({xScale(1)},0)">
      <text y={height - padding.bottom + 36} >Recall target</text>
    </g>
	</g>

	<!-- data -->
	{#each points as point}
		<circle cx={xScale(point.x)} cy={yScale(point.y)} r="5" />
	{/each}
</svg>

<style>
	svg {
		width: 100%;
		height: 100%;
		float: left;
	}

	circle {
		fill: orange;
		fill-opacity: 0.6;
		stroke: rgba(0, 0, 0, 0.5);
	}

	.tick line {
		stroke: #ddd;
		stroke-dasharray: 2;
	}

	text {
		font-size: 12px;
		fill: #999;
	}

	.x-axis text {
		text-anchor: middle;
	}

	.y-axis text {
		text-anchor: end;
	}
</style>
