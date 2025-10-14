<!-- TO-DO: Create a station comparison dashboard -->
<!-- ✅ Create grid layout -->
<!-- ✅ Add shared time axis -->
<!-- ✅ Add individual Y-axes -->
<!-- ✅ Calculate monthly data per station -->
<!-- ✅ Draw line charts -->
<!-- ✅ Add station labels -->
<!-- ⬜ Add brushing -->
<!-- ✅ Add tooltips -->
<!-- ⬜ Add toggle button -->
<!-- ⬜ Add click to focus -->
<!-- ✅ Add legends -->
<!-- ✅ Style and refine -->
<!-- ⬜ Add annotations -->

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

	// CONSTANTS AND CONFIGURATION
	
	// Dashboard dimensions - BIGGER!
	const dashboardWidth = 1485;
	const dashboardHeight = 1000;
	
	// Grid layout: 2 rows × 4 columns
	const cols = 4;
	const rows = 2;
	
	// Chart dimensions for each small chart - MORE SPACE!
	const chartMargin = { top: 40, right: 30, bottom: 50, left: 60 };
	const chartWidth = (dashboardWidth / cols) - 30;
	const chartHeight = (dashboardHeight / rows) - 30;
	
	const innerWidth = chartWidth - chartMargin.left - chartMargin.right;
	const innerHeight = chartHeight - chartMargin.top - chartMargin.bottom;

	// AQI level definitions with colors
	// Source: Assignment description
	const aqiLevels = [
		{ name: 'Good', min: 0, max: 50, color: '#9cd84e' },
		{ name: 'Moderate', min: 51, max: 100, color: '#facf39' },
		{ name: 'Unhealthy for Sensitive Groups', min: 101, max: 150, color: '#f99049' },
		{ name: 'Unhealthy', min: 151, max: 200, color: '#f65e5f' },
		{ name: 'Very Unhealthy', min: 201, max: 300, color: '#a070b6' },
		{ name: 'Hazardous', min: 301, max: 500, color: '#a06a7b' }
	];

	// Get unique station names
	// Source: Array.from with Set for unique values
	const stations = Array.from(new Set(data.map(d => d.stationName)));

	// STATE VARIABLES

	// Tooltip state
	let hoveredPoint = $state<{station: string, date: Date, avgAqi: number, avgPm25: number} | null>(null);
	let tooltipX = $state(0);
	let tooltipY = $state(0);

	// DERIVED DATA

	// Calculate monthly data for each station
	// Source: D3 rollup for grouping and aggregation
	const monthlyDataByStation = $derived(
		Array.from(
			d3.group(data, d => d.stationName),
			([station, stationData]) => ({
				station,
				monthlyData: Array.from(
					d3.rollup(
						stationData,
						(v) => ({
							avgAqi: d3.mean(v, d => d.usAqi) || 0,
							avgPm25: d3.mean(v, d => d.pm25) || 0,
							count: v.length
						}),
						(d) => d3.timeMonth.floor(d.timestamp)
					),
					([date, stats]) => ({
						date,
						...stats
					})
				).sort((a, b) => a.date.getTime() - b.date.getTime())
			})
		)
	);

	// SCALES

	// Get the full time extent across all stations
	// Source: D3 extent for min/max values
	const timeExtent = $derived(
		d3.extent(data, d => d.timestamp) as [Date, Date]
	);

	// Get the max AQI and PM2.5 values for scale domains
	const maxAqi = $derived(d3.max(data, d => d.usAqi) || 300);
	const maxPm25 = $derived(d3.max(data, d => d.pm25) || 100);

	// Create scale functions (we'll use these for each chart)
	// Source: D3 scale documentation
	function createXScale() {
		return d3.scaleTime()
			.domain(timeExtent)
			.range([0, innerWidth]);
	}

	function createYScaleAqi() {
	return d3.scaleLinear()
		.domain([0, d3.max(data, d => d.usAqi) ?? 300])  // Use actual max from data
		.range([innerHeight, 0]);
}

	function createYScalePm25() {
		return d3.scaleLinear()
			.domain([0, maxPm25])
			.range([innerHeight, 0]);
	}

	// Function to get AQI color
	// Source: Same as AQI Chart
	function getAQIColor(aqi: number): string {
		for (const level of aqiLevels) {
			if (aqi >= level.min && aqi <= (level.max || Infinity)) {
				return level.color;
			}
		}
		return aqiLevels[aqiLevels.length - 1].color;
	}

// Render axes using $effect
// Source: D3 axis rendering
// Render axes using $effect
// Source: D3 axis rendering
$effect(() => {
	stations.forEach(station => {
		const xAxisEl = document.getElementById(`x-axis-${station}`);
		const yAxisEl = document.getElementById(`y-axis-${station}`);
		
		if (xAxisEl && yAxisEl) {
			const xScale = createXScale();
			const yScale = createYScaleAqi();
			
			const xAxis = d3.axisBottom(xScale)
				.ticks(d3.timeYear.every(2))  // Show every 2 years
				.tickFormat(d3.timeFormat('%Y') as any);  // Just year (2016, 2018, 2020, 2022)
			
			const yAxis = d3.axisLeft(yScale)
				.ticks(5);
			
			d3.select(xAxisEl)
				.call(xAxis as any)
				.selectAll('text')
				.style('font-size', '11px');  // Slightly bigger font
				
			d3.select(yAxisEl).call(yAxis as any);
		}
	});
});

$effect(() => {
	console.log('Time extent:', timeExtent);
	console.log('First date:', d3.min(data, d => d.timestamp));
	console.log('Last date:', d3.max(data, d => d.timestamp));
});

</script>

<div class="dashboard">
	<h2>Station Comparison Dashboard</h2>
	<p class="subtitle">Monthly average AQI across Pittsburgh monitoring stations</p>
	
	<svg width={dashboardWidth} height={dashboardHeight}>
		<!-- Draw grid of charts -->
		{#each stations as station, i}
			{@const col = i % cols}
			{@const row = Math.floor(i / cols)}
			{@const x = col * (chartWidth + 30)}
			{@const y = row * (chartHeight + 30)}
			
			<g transform="translate({x}, {y})">
				<!-- Chart background -->
				<rect 
					width={chartWidth} 
					height={chartHeight} 
					fill="white" 
					stroke="#ddd" 
					stroke-width="1"
					rx="4"
				/>
				
				<!-- Station name label -->
				<text 
					x={chartWidth / 2} 
					y={25} 
					text-anchor="middle" 
					font-weight="bold"
					font-size="15"
				>
					{station}
				</text>
				
				<!-- Chart area -->
				<g transform="translate({chartMargin.left}, {chartMargin.top})">
					{#each monthlyDataByStation.filter(s => s.station === station) as stationData}
						{@const xScale = createXScale()}
						{@const yScale = createYScaleAqi()}
						
						<!-- Background grid lines -->
						{#each yScale.ticks(5) as tick}
							<line
								x1="0"
								y1={yScale(tick)}
								x2={innerWidth}
								y2={yScale(tick)}
								stroke="#f0f0f0"
								stroke-width="1"
							/>
						{/each}
						
						<!-- Draw line segments colored by AQI -->
						{#each stationData.monthlyData.slice(0, -1) as point, idx}
							{@const nextPoint = stationData.monthlyData[idx + 1]}
							{@const color = getAQIColor(point.avgAqi)}
							<line
								x1={xScale(point.date)}
								y1={yScale(point.avgAqi)}
								x2={xScale(nextPoint.date)}
								y2={yScale(nextPoint.avgAqi)}
								stroke={color}
								stroke-width="2"
							/>
						{/each}
						
						<!-- Draw dots at each data point -->
						{#each stationData.monthlyData as point}
							{@const color = getAQIColor(point.avgAqi)}
							<circle
								cx={xScale(point.date)}
								cy={yScale(point.avgAqi)}
								r="2.5"
								fill={color}
								style="cursor: pointer;"
								role="button"
								tabindex="0"
								onmouseenter={(e) => {
									hoveredPoint = {
										station: station,
										date: point.date,
										avgAqi: point.avgAqi,
										avgPm25: point.avgPm25
									};
									const svgRect = e.currentTarget.ownerSVGElement?.getBoundingClientRect();
									const circleRect = e.currentTarget.getBoundingClientRect();
									if (svgRect) {
										tooltipX = circleRect.left - svgRect.left + circleRect.width / 2;
										tooltipY = circleRect.top - svgRect.top;
									}
								}}
								onmouseleave={() => {
									hoveredPoint = null;
								}}
							/>
						{/each}
						
						<!-- Axes - using id instead of bind -->
						<g 
							class="x-axis" 
							id="x-axis-{station}"
							transform="translate(0, {innerHeight})"
						></g>
						<g 
							class="y-axis"
							id="y-axis-{station}"
						></g>
					{/each}
				</g>
			</g>
		{/each}
		
		<!-- Tooltip inside SVG -->
		{#if hoveredPoint}
			<g transform="translate({tooltipX}, {tooltipY - 10})" pointer-events="none">
				<rect
					x="-70"
					y="-75"
					width="140"
					height="65"
					fill="white"
					stroke="#333"
					stroke-width="1"
					rx="4"
					
				/>
                <text x="0" y="-55" text-anchor="middle" font-size="12">
					{hoveredPoint.date.toLocaleDateString('en-US', { month: 'short', year: 'numeric' })}
				</text>
				<text x="0" y="-38" text-anchor="middle" font-size="12">
					AQI: {hoveredPoint.avgAqi.toFixed(1)}
				</text>
				<text x="0" y="-22" text-anchor="middle" font-size="12">
					PM2.5: {hoveredPoint.avgPm25.toFixed(1)} μg/m³
				</text>
			</g>
		{/if}
		
		<!-- Shadow filter -->
		<defs>
			<filter id="shadow">
				<feDropShadow dx="0" dy="2" stdDeviation="4" flood-opacity="0.3"/>
			</filter>
		</defs>
	</svg>
	
	<!-- Legend -->
	<div class="legend">
		<h3>AQI Levels</h3>
		<div class="legend-items">
			{#each aqiLevels as level}
				<div class="legend-item">
					<div class="legend-color" style="background-color: {level.color}"></div>
					<span>{level.name} ({level.min}-{level.max ?? '500+'})</span>
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	.dashboard {
		padding: 20px;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
		position: relative;
	}
	
	h2 {
		margin-bottom: 5px;
	}
	
	.subtitle {
		color: #666;
		margin-bottom: 20px;
		font-size: 14px;
	}
	
	:global(.x-axis text),
	:global(.y-axis text) {
		font-size: 10px;
	}
	
	:global(.x-axis path),
	:global(.y-axis path),
	:global(.x-axis line),
	:global(.y-axis line) {
		stroke: #999;
	}
	
	.legend {
		margin-top: 30px;
		padding: 20px;
		background: white;
		border: 1px solid #ddd;
		border-radius: 8px;
	}
	
	.legend h3 {
		margin: 0 0 15px 0;
		font-size: 16px;
	}
	
	.legend-items {
		display: flex;
		gap: 20px;
		flex-wrap: wrap;
	}
	
	.legend-item {
		display: flex;
		align-items: center;
		gap: 8px;
	}
	
	.legend-color {
		width: 30px;
		height: 15px;
		border-radius: 3px;
	}
	
	.legend-item span {
		font-size: 13px;
	}
</style>