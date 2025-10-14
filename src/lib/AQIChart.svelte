<!-- TO-DO: Create a time-series visualization of the air quality data. -->
<!-- ✅ Show the monthly average air quality (AQI) as a line. -->
<!-- ✅ Show the inner 80 percentiles (10% to 90%) as an area behind the line. -->
<!-- ✅ Align the date with the 15th day of the month. -->
<!-- ✅ In the background, show the AQI levels as color. -->
<!-- ✅ Add a dropdown to select the station. -->
<!-- ✅ Sort the station names by count. -->
<!-- ✅ If no station is selected, show the series for all stations. -->
<!-- ✅ Add a checkbox to toggle showing the raw data as points (time and AQI) on the chart. -->
<!-- ✅ Add key for AQI levels -->

<script lang="ts">
	import * as d3 from 'd3';

	// TYPE DEFINITIONS

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

	// PROPS
	const { data }: { data: Item[] } = $props();

	// STATE VARIABLES

	// Dropdown selection for filtering by station (null = all stations)
	let selectedStation = $state<string | null>(null);

	// Checkbox toggle for showing raw data points
	let showRawData = $state(false);

	// CONSTANTS AND CONFIGURATION

	// Chart Dimensions
	// Source: https://d3js.org/getting-started#d3-in-svelte
	const width = $state(800);
	const height = $state(400);
	let margin = $state({ top: 20, right: 20, bottom: 60, left: 40 });

	// AQI level definitions with colors
	// Source: Assignment description
	const aqiLevels = [
		{ name: 'Good', min: 0, max: 50, color: '#9cd84e' },
		{ name: 'Moderate', min: 51, max: 100, color: '#facf39' },
		{ name: 'Unhealthy for Sensitive Groups', min: 101, max: 150, color: '#f99049' },
		{ name: 'Unhealthy', min: 151, max: 200, color: '#f65e5f' },
		{ name: 'Very Unhealthy', min: 201, max: 300, color: '#a070b6' },
		{ name: 'Hazardous', min: 301, color: '#a06a7b' }
	];

	// DERIVED DATA

	// Count and sort stations by number of records (descending)
	const stationCounts = $derived(
		Array.from(
			d3.rollup(
				data,
				(v) => v.length,
				(d) => d.stationName
			),
			([name, count]) => ({ name, count })
		).sort((a, b) => b.count - a.count)
	);

	// Filter data by selected station, or show all if none selected
	const filteredData = $derived(
		selectedStation ? data.filter((d) => d.stationName == selectedStation) : data
	);

	// Calculate monthly statistics for each station
	// Returns array of objects, each containing station name and its monthly data
	const monthlyDataByStation = $derived.by(() => {
		if (selectedStation) {
			// Single station mode: return one series
			return [
				{
					station: selectedStation,
					data: calculateMonthlyStats(filteredData)
				}
			];
		} else {
			// All stations mode: return multiple series
			return Array.from(
				d3.group(
					data.filter((d) => d.stationName),
					(d) => d.stationName
				),
				([station, stationData]) => ({
					station,
					data: calculateMonthlyStats(stationData)
				})
			);
		}
	});

	// Helper function to calculate monthly statistics
	// Groups data by month and computes mean, 10th percentile, and 90th percentile
	function calculateMonthlyStats(dataPoints: Item[]) {
		return Array.from(
			d3.rollup(
				dataPoints,
				(v) => {
					const aqiVals = v.map((d) => d.usAqi).sort(d3.ascending);
					return {
						mean: d3.mean(aqiVals), // Monthly mean
						p10: d3.quantile(aqiVals, 0.1), // 10th percentile
						p90: d3.quantile(aqiVals, 0.9) // 90th percentile
					};
				},
				(d) => d3.timeMonth.floor(d.timestamp)
			),
			([date, stats]) => ({
				// Align dates to 15th of each month
				date: new Date(date.getFullYear(), date.getMonth(), 15),
				...stats
			})
		);
	}

	// SCALES

	// X-axis: Time scale spanning all months in the dataset
	// Source: Professor's GitHub Repo
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

	// Y-axis: linear scale from 0 to max AQI value
	// Source: Professor's GitHub Repo, FullSVGBarChart.svelte
	let yScale = $derived(
		d3
			.scaleLinear()
			.range([height - margin.bottom, margin.top])
			.domain([0, d3.max(filteredData, (d) => d.usAqi) ?? 300])
	);

	// Color scale for differentiating stations when showing all
	const colorScale = $derived(
		d3.scaleOrdinal(d3.schemeCategory10).domain(stationCounts.map((s) => s.name))
	);

	// LINE AND AREA GENERATORS

	// Line generator for monthly mean air quality (AQI)
	let line = $derived(
		d3
			.line<{ date: Date; mean: number | undefined }>()
			.x((d) => xScale(d.date))
			.y((d) => yScale(d.mean ?? 0))
	);

	// Area generator for 10th-90th percentile range
	// Source: AI fixed errors with type annotations
	let area = $derived(
		d3
			.area<{ date: Date; p10: number | undefined; p90: number | undefined }>()
			.x((d) => xScale(d.date))
			.y0((d) => yScale(d.p10 ?? 0))
			.y1((d) => yScale(d.p90 ?? 0))
	);

	// AXES

	// Source: Professor's GitHub Repo, FullSVGBarChart.svelte
	// Source: 
	let xAxis = $derived(d3.axisBottom(xScale).tickFormat(d3.timeFormat('%b %Y') as any));
	let yAxis = $derived(d3.axisLeft(yScale));

	let xAxisRef: SVGGElement;
	let yAxisRef: SVGGElement;

	// Render axes when data loads
	$effect(() => {
		if (xAxisRef && data.length > 0) { 
			d3.select(xAxisRef)
				.call(xAxis)
				.selectAll('text')
				.attr('transform', 'rotate(-45)') // Slant the axis
				.style('text-anchor', 'end')
				.attr('dx', '-.8em')
				.attr('dy', '.15em');
		}
	});

	$effect(() => {
		if (yAxisRef && data.length > 0) {
			d3.select(yAxisRef).call(yAxis);
		}
	});
</script>

<!-- CONTROLS -->

<!-- Station selection dropdown -->
<select bind:value={selectedStation}>
	<option value={null}> All Stations ({data.length})</option>
	{#each stationCounts as station}
		<option value={station.name}>
			{station.name} ({station.count})
		</option>
	{/each}
</select>

<br />

<!-- Raw data toggle checkbox -->
<!-- Source: https://svelte.dev/docs/element-directives#bind -->
<label>
	Show Raw Data
	<input type="checkbox" bind:checked={showRawData} />
</label>

<!-- Display filtered record count -->
<!-- svelte-ignore a11y_label_has_associated_control -->
<label>
	Number of Records: {filteredData.length}
</label>

<br />

<div class="chart-wrapper">
<svg {width} {height}>
	<!-- AQI level background bands -->
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
		<!-- Show percentile area only for single station -->
		{#if selectedStation}
			<path d={area(stationSeries.data)} fill="grey" opacity="0.25" />
		{/if}

		<!-- Monthly mean line; colored by station when showing all -->
		<path
			d={line(stationSeries.data)}
			fill="none"
			stroke={selectedStation ? 'black' : colorScale(stationSeries.station)}
			stroke-width="2"
		/>
	{/each}

	<!-- Raw data points (when checkbox is enabled) -->
	{#if showRawData}
		{#each filteredData as d}
			<circle cx={xScale(d.timestamp)} cy={yScale(d.usAqi)} r="1" fill="black" />
		{/each}
	{/if}

	<!-- Axes -->
	<!-- Source: Professor's GitHub Repo, FullSVGBarChart.svelte -->
	<g class="x-axis" transform="translate(0,{height - margin.bottom})" bind:this={xAxisRef}></g>
	<g class="y-axis" transform="translate({margin.left},0)" bind:this={yAxisRef}></g>
</svg>


<!-- LEGENDS -->
<div class="legends-container">
	<!-- AQI Level Legend -->
	<div class="legend-section">
		<h4>Air Quality Index Levels</h4>
		{#each aqiLevels as level}
			<div class="legend-item">
				<div class="legend-color" style="background-color: {level.color};"></div>
				<div class="legend-text">
					<div class="legend-name">{level.name}</div>
					<div class="legend-range">{level.min}-{level.max ?? '301+'}</div>
				</div>
			</div>
		{/each}
	</div>

	<!-- Station Legend (only visible when showing all stations) -->
	{#if !selectedStation}
		<div class="legend-section">
			<h4>Stations</h4>
			{#each stationCounts as station}
				<div class="legend-item">
					<div class="legend-line" style="background-color: {colorScale(station.name)};"></div>
					<div class="legend-text">
						<div class="legend-name">{station.name}</div>
					</div>
				</div>
			{/each}
		</div>
	{/if}
</div>
</div>

<style>
	* {
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
	}
	
	svg {
		border: 1px solid #e0e0e0;
		border-radius: 8px;
	}
	
	select {
		border: 1px solid #ccc;
		border-radius: 8px;
		font-size: 14px;
	}
	
	label {
		font-size: 14px;
		display: flex;
		align-items: center;
		padding-top: 5px;
	}

	.chart-wrapper {
		display: flex;
		gap: 30px;
		align-items: flex-start;
	}

	.legends-container {
		display: flex;
		flex-direction: column;
		gap: 20px;
		min-width: 200px;
		max-height: 400px;
	}

	.legend-section {
		display: flex;
		flex-direction: column;
		gap: 5px;
	}

	.legend-section h4 {
		font-size: 14px;
		margin: 0 0 5px 0;
		color: #333;
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 10px;
		font-size: 12px;
	}

	.legend-color {
		width: 20px;
		height: 20px;
		border-radius: 4px;
		flex-shrink: 0;
		opacity: 0.5;
	}

	.legend-line {
		width: 20px;
		height: 3px;
		border-radius: 2px;
		flex-shrink: 0;
	}

	.legend-text {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	.legend-name {
		color: #333;
		line-height: 1.1;
	}

	.legend-range {
		font-size: 10px;
		color: #666;
	}
</style>
