<!-- TO-DO: Create a time-series visualization of the air quality data. -->
<!-- ✅ Step 1: Show the monthly average air quality (AQI) as a line. -->
<!-- ✅ Step 2: Show the inner 80 percentiles (10% to 90%) as an area behind the line. -->
<!-- ✅ Step 3: Add a dropdown to select the station. -->
<!-- ✅ Step 4: If no station is selected, the chart should show the series for all stations. -->
<!-- ✅ Step 5: Sort the station names by count. -->
<!-- ✅ Step 6: Align the date with the 15th day of the month. -->
<!-- ✅ Step 7: In the background of the chart, show the AQI levels as color. -->
<!-- ✅ Step 8: Add a checkbox to toggle showing the raw data as points (time and AQI) on the chart. -->
<!-- Step 9: Add key for AQI Levels! -->

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

	// Step 8: Add a checkbox to toggle showing the raw data as points
	let showRawData = $state(false);

	// Step 5: Sort the station names by count.
	const stationCounts = $derived(
		Array.from(
			d3.rollup(
				data,
				(v) => v.length,
				(d) => d.stationName
			),
			([name, count]) => ({ name, count })
		).sort((a, b) => b.count - a.count) // sort by descending count
	);

	// Step 3: Add a dropdown to select station
	let selectedStation = $state<string | null>(null);

	// Step 4: Filter data by selected station (or show it all if null)
	const filteredData = $derived(
		selectedStation ? data.filter((d) => d.stationName == selectedStation) : data
	);

	// Step 7: In the background of the chart, show the AQI levels as color.
	// given in assignment
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

	// group by month and calculate mean AQI
	// Step 1: Show the monthly average air quality (AQI) as a line.
	// Step 2: Show the inner 80 percentiles (10% to 90%) as an area behind the line.
	// Step 4: If no station is selected, show series for all stations group by month and calculate mean AQI
	const monthlyDataByStation = $derived.by(() => {
		if (selectedStation) {
			// single station selected -> return array with one entry !! this took so long
			return [
				{
					station: selectedStation,
					data: Array.from(
						d3.rollup(
							// groups data and applies an aggregation function to each group
							filteredData,
							(v) => {
								// reducer function; v is the array of all items in one group / month
								const aqiVals = v.map((d) => d.usAqi).sort(d3.ascending); // extracts the usAqi values from each item and sorts values from low to high (u need to do this to use quartile)
								return {
									mean: d3.mean(aqiVals), // calculates the mean of the array
									p10: d3.quantile(aqiVals, 0.1), // returns the value at a given percentile (0.10 = 10th percentile and then 0.90 = 90th percentile)
									p90: d3.quantile(aqiVals, 0.9)
								};
							},
							// https://d3js.org/d3-time#timeMonth
							(d) => d3.timeMonth.floor(d.timestamp) // data is being grouped by month; rounds date down to first day of the month
						),
						([date, stats]) => ({
							// Step 6: Align the date with the 15th day of the month.
							date: new Date(date.getFullYear(), date.getMonth(), 15),
							...stats // the spread operator (from mdn documentation) copies the properties from one object into another
							// without operator it creates: { date: "2021-08-15", stats: { mean: 42, p10: 35, p90: 48 } }
							// with operator it creates: { date: "2021-08-15", mean: 42, p10: 35, p90: 48 }
						})
					)
				}
			];
		} else {
			// no station selected - show all stations as separate series
			return Array.from(
				d3.group(
					data.filter((d) => d.stationName), // filter out any records with undefined station names
					(d) => d.stationName
				),
				// transform each group (station) into an object with station name and monthly data
				([station, stationData]) => ({
					station, // station name
					data: Array.from(
						d3.rollup(
							stationData, // all the data points for this specific station
							(v) => {
								// for each month, calculate statistics from all AQI values in that month
								const aqiVals = v.map((d) => d.usAqi).sort(d3.ascending);
								return {
									mean: d3.mean(aqiVals),
									p10: d3.quantile(aqiVals, 0.1),
									p90: d3.quantile(aqiVals, 0.9)
								};
							},
							(d) => d3.timeMonth.floor(d.timestamp)
						),
						([date, stats]) => ({
							date: new Date(date.getFullYear(), date.getMonth(), 15),
							...stats
						})
					)
				})
			);
		}
	});
	// looks like this when I console.log
	// monthly data: [
	//   {
	//     "date": "2021-08-15T04:00:00.000Z",
	//     "aqi": 43
	//   },

	// colored lines for different stations
	const colorScale = $derived(
		d3.scaleOrdinal(d3.schemeCategory10).domain(stationCounts.map((s) => s.name))
	);

	// adopted from lecture
	let xScale = $derived(
		d3
			.scaleTime()
			.range([margin.left, width - margin.right])
			.domain(
				d3.extent(
					monthlyDataByStation.flatMap((s) => s.data),
					(d) => d.date
				) as [Date, Date]
			)
	);

	// adopted from lecture
	let yScale = $derived(
		d3
			.scaleLinear()
			.range([height - margin.bottom, margin.top])
			.domain([0, d3.max(filteredData, (d) => d.usAqi) ?? 300])
	);

	// create line
	// Step 1: Show the monthly average air quality (AQI) as a line.
	let line = $derived(
		d3
			.line<{ date: Date; mean: number | undefined }>()
			.x((d) => xScale(d.date))
			.y((d) => yScale(d.mean ?? 0))
	);

	// keep getting errors so i'm getting help with type annotations
	let area = $derived(
		d3
			.area<{ date: Date; p10: number | undefined; p90: number | undefined }>()
			.x((d) => xScale(d.date))
			.y0((d) => yScale(d.p10 ?? 0))
			.y1((d) => yScale(d.p90 ?? 0))
	);

	// adopted from Professor's repository: FullSVGBarChart.svelte
	let xAxis = $derived(d3.axisBottom(xScale).tickFormat(d3.timeFormat('%Y') as any));
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

<!-- dropdown -->
<select bind:value={selectedStation}>
	<option value={null}> All Stations ({data.length})</option>
	{#each stationCounts as station}
		<option value={station.name}>
			{station.name} ({station.count})
		</option>
	{/each}
</select>

<br />

<!-- Step 8: Checkbox to toggle raw data -->
<!-- https://svelte.dev/docs/element-directives#bind -->
<label>
	Show Raw Data
	<input type="checkbox" bind:checked={showRawData} />
</label>

<br />

<label>
	Number of Records: {filteredData.length}
</label>

<br />

<svg {width} {height}>
	<!-- Step 7: In the background of the chart, show the AQI levels as color. -->
	{#each aqiLevels as level}
		<rect
			x={margin.left}
			y={Math.max(margin.top, yScale(level.max ?? 500))}
			width={width - margin.left - margin.right}
			height={Math.max(
				0,
				Math.min(yScale(level.min), height - margin.bottom) -
					Math.max(margin.top, yScale(level.max ?? 500))
			)}
			fill={level.color}
			opacity="0.5"
		/>
	{/each}

	<!-- Draw line and area for each station -->
	{#each monthlyDataByStation as stationSeries}
		<!-- Step 2: Show the inner 80 percentiles (10% to 90%) as an area behind the line (only for single station) -->
		{#if selectedStation}
			<path d={area(stationSeries.data)} fill="grey" opacity="0.25" />
		{/if}

		<!-- Step 1: Show the monthly average air quality (AQI) as a line -->
		<path
			d={line(stationSeries.data)}
			fill="none"
			stroke={selectedStation ? 'black' : colorScale(stationSeries.station)}
			stroke-width="2"
		/>
	{/each}

	<!-- Step 8: Show raw data points when checkbox is checked -->
	{#if showRawData}
		{#each filteredData as d}
			<circle cx={xScale(d.timestamp)} cy={yScale(d.usAqi)} r="1" fill="black" />
		{/each}
	{/if}

	<!-- adopted from Professor's repository: FullSVGBarChart.svelte -->
	<g class="x-axis" transform="translate(0,{height - margin.bottom})" bind:this={xAxisRef}></g>
	<g class="y-axis" transform="translate({margin.left},0)" bind:this={yAxisRef}></g>
</svg>

<!-- <pre>
	{JSON.stringify(data[0], null, 2)}
</pre> -->

<br />

<!-- this legend will show up on the all stations option -->
{#if !selectedStation}
	<div style="margin-top: 20px;">
		<div style="display: flex; flex-wrap: wrap; gap: 15px; margin-top: 10px;">
			{#each stationCounts as station}
				<div style="display: flex; align-items: center; gap: 5px;">
					<div
						style="width: 20px; height: 3px; background-color: {colorScale(station.name)};"
					></div>
					<span style="font-size: 14px;">{station.name}</span>
				</div>
			{/each}
		</div>
	</div>
{/if}

<!-- Step 9: Add key for AQI Levels -->
<div style="margin-top: 20px;">
	<div style="display: flex; align-items: center; gap: 10px; flex-wrap: wrap">
		<label style="margin-right: 10px;">US AQI</label>
		{#each aqiLevels as level}
			<div 
				style="
					display: flex; 
					align-items: center; 
					justify-content: center;
					background-color: {level.color}80; 
					padding: 12px 20px; 
					border-radius: 12px;
					min-width: 120px;
				"
			>
				<div style="text-align: center">
					<div style="font-size: 13px; line-height: 1.3;">{level.name}</div>
					<div style="font-size: 12px;">{level.min}-{level.max ?? '301+'}</div>
				</div>
			</div>
		{/each}
	</div>
</div>

<style>
	svg {
		font-family: sans-serif;
	}
</style>
