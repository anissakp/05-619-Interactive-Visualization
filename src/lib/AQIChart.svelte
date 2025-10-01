<!-- TO-DO: Create a time-series visualization of the air quality data. -->
<!-- Step 1: Show the monthly average air quality (AQI) as a line. -->
<!-- Step 2: Show the inner 80 percentiles (10% to 90%) as an area behind the line. -->
<!-- Step 3: Add a dropdown to select the station. -->
<!-- Step 4: If no station is selected, the chart should show the series for all stations. -->
<!-- Step 5: Sort the station names by count. -->
<!-- Step 6: Align the date with the 15th day of the month. -->
<!-- Step 7: In the background of the chart, show the AQI levels as color. -->
<!-- Step 8: Add a checkbox to toggle showing the raw data as points (time and AQI) on the chart. -->

<script lang="ts">
	import * as d3 from 'd3';

	// typescript interface that describes structure of each data item
	interface Item {
		city: string;
		country: string;
		mainPollutant: string;
		pm25: number;
		state: string;
		stationName: string;
		timestamp: Date;
		usAqi: number;
	}

	// properties this component accepts
	const { data }: { data: Item[] } = $props();

	// Step 7: In the background of the chart, show the AQI levels as color.
	const aqiLevels = [
		{ name: 'Good', min: 0, max: 50, color: '#9cd84e' },
		{ name: 'Moderate', min: 51, max: 100, color: '#facf39' },
		{ name: 'Unhealthy for Sensitive Groups', min: 101, max: 150, color: '#f99049' },
		{ name: 'Unhealthy', min: 151, max: 200, color: '#f65e5f' },
		{ name: 'Very Unhealthy', min: 201, max: 300, color: '#a070b6' },
		{ name: 'Hazardous', min: 301, color: '#a06a7b' }
	];

	// chart dimensions
	// adopted from https://d3js.org/getting-started#d3-in-svelte
	const width = $state(800);
	const height = $state(400);
	let margin = $state({ top: 20, right: 20, bottom: 30, left: 40 });

	// Step 1: Show the monthly average air quality (AQI) as a line.
	// group by month and calculate mean AQI
	const monthlyData = $derived(
		Array.from(
			d3.rollup(
				// group and reduce values
				data,
				(v) => d3.mean(v, (d) => d.usAqi), // calculates the mean of the array
				(d) => d3.timeMonth.floor(d.timestamp) // data is being grouped by month
			),
			([date, aqi]) => ({
				// Step 6: Align the date with the 15th day of the month.
				date: new Date(date.getFullYear(), date.getMonth(), 15),
				aqi
			})
		)
	);
	// looks like this when I console.log
	// monthly data: [
	//   {
	//     "date": "2021-08-15T04:00:00.000Z",
	//     "aqi": 43
	//   },

	// adopted from lecture
	let xScale = $derived(
		d3
			.scaleTime()
			.range([margin.left, width - margin.right])
			.domain(d3.extent(monthlyData, (d) => d.date) as [Date, Date])
	);

	// adopted from lecture
	let yScale = $derived(
		d3
			.scaleLinear()
			.range([height - margin.bottom, margin.top])
			.domain([0, d3.max(data, (d) => d.usAqi) ?? 300])
	);

	// create line generator
	let lineGenerator = $derived(
		d3
			.line<{ date: Date; aqi: number | undefined }>()
			.x((d) => xScale(d.date))
			.y((d) => yScale(d.aqi ?? 0))
	);

	// adopted from lecture
	let xAxis = $derived(d3.axisBottom(xScale).tickFormat(d3.timeFormat('%b') as any));
	let yAxis = $derived(d3.axisLeft(yScale));

	let xAxisRef: SVGGElement;
	let yAxisRef: SVGGElement;

	$effect(() => {
		if (xAxisRef && data.length > 0) {
			d3.select(xAxisRef).call(xAxis);
		}
	});

	$effect(() => {
		if (yAxisRef && data.length > 0) {
			d3.select(yAxisRef).call(yAxis);
		}
	});

	// just for debugging; can be removed
	$inspect(data);
</script>

<svg {width} {height}>
	<path d={lineGenerator(monthlyData)} fill="none" stroke="black" stroke-width="2" />
	<g class="x-axis" transform="translate(0,{height - margin.bottom})" bind:this={xAxisRef}></g>
	<g class="y-axis" transform="translate({margin.left},0)" bind:this={yAxisRef}></g>
</svg>

<pre>
	{JSON.stringify(data[0], null, 2)}
</pre>

<style>
	svg {
		font-family: sans-serif;
	}
</style>
