<!-- src/lib/RankCompareChart.svelte -->
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

	// STATE
	let selectedStations = $state<string[]>([]);

	// AQI categories
	const aqiCategories = [
		{ name: 'Good', min: 0, max: 50, color: '#9cd84e' },
		{ name: 'Moderate', min: 51, max: 100, color: '#facf39' },
		{ name: 'Unhealthy for Sensitive Groups', min: 101, max: 150, color: '#f99049' }
	];

	function getCategory(aqi: number) {
		for (let cat of aqiCategories) {
			if (aqi <= cat.max) return cat;
		}
		return aqiCategories[aqiCategories.length - 1];
	}

	// DERIVED DATA - Calculate station statistics
	const stationStats = $derived.by(() => {
		return Array.from(
			d3.group(data, (d) => d.stationName),
			([station, records]) => {
				const aqiValues = records.map((d) => d.usAqi);
				const goodDays = records.filter((d) => d.usAqi <= 50).length;
				const moderateDays = records.filter((d) => d.usAqi > 50 && d.usAqi <= 100).length;
				const unhealthyDays = records.filter((d) => d.usAqi > 100).length;

				return {
					station,
					totalDays: records.length,
					avgAqi: d3.mean(aqiValues) || 0,
					medianAqi: d3.median(aqiValues) || 0,
					maxAqi: d3.max(aqiValues) || 0,
					goodDays,
					moderateDays,
					unhealthyDays,
					goodPercent: (goodDays / records.length) * 100,
					moderatePercent: (moderateDays / records.length) * 100,
					unhealthyPercent: (unhealthyDays / records.length) * 100
				};
			}
		).sort((a, b) => b.goodPercent - a.goodPercent);
	});

	const selectedStationData = $derived(
		stationStats.filter((s) => selectedStations.includes(s.station))
	);

	// HELPER FUNCTIONS
	// Toggle station selection (max 2 stations for comparison)
	function toggleStation(station: string) {
		if (selectedStations.includes(station)) {
			selectedStations = selectedStations.filter((s) => s !== station);
		} else {
			if (selectedStations.length < 2) {
				selectedStations = [...selectedStations, station];
			} else {
				selectedStations = [selectedStations[1], station];
			}
		}
	}
</script>

<!-- MAIN CONTAINER -->
<div class="container">
	<!-- svelte-ignore a11y_label_has_associated_control -->
	<p>
		Find the best neighborhood for people with respiratory issues such as asthma.
		<br />
		Click on any neighborhood bar to see details. Click two neighborhoods to compare side-by-side.
	</p>

	<!-- RANKINGS -->
	<h4>Rankings: Click to Select (up to 2 for comparison)</h4>

	<div class="rankings-wrapper">
		<div class="rankings-list">
			{#each stationStats as station, index}
				{@const isSelected = selectedStations.includes(station.station)}

				<!-- svelte-ignore a11y_click_events_have_key_events -->
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div
					class="ranking-item"
					class:selected={isSelected}
					onclick={() => toggleStation(station.station)}
				>
					<div class="ranking-content">
						<div class="rank-number">
							{index + 1}
						</div>

						<div class="station-info">
							<div class="station-name">{station.station}</div>

							<!-- Bar Chart -->
							<!-- Displays percentage distribution of AQI categories -->
							<div class="bar-container">
								<div
									class="bar-segment bar-good"
									style="width: {station.goodPercent}%"
									title="{station.goodDays} good days"
								>
									{#if station.goodPercent > 10}
										{station.goodPercent.toFixed(0)}%
									{/if}
								</div>
								<div
									class="bar-segment bar-moderate"
									style="width: {station.moderatePercent}%"
									title="{station.moderateDays} moderate days"
								>
									{#if station.moderatePercent > 10}
										{station.moderatePercent.toFixed(0)}%
									{/if}
								</div>
								<div
									class="bar-segment bar-unhealthy"
									style="width: {station.unhealthyPercent}%"
									title="{station.unhealthyDays} unhealthy days"
								>
									{#if station.unhealthyPercent > 10}
										{station.unhealthyPercent.toFixed(0)}%
									{/if}
								</div>
							</div>

							<!-- Stats -->
							<!-- Summary metrics for each station -->
							<div class="station-stats">
								<span>Avg AQI: <strong>{station.avgAqi.toFixed(0)}</strong></span>
								<span>Good Days: <strong>{station.goodDays}</strong></span>
								<span>Unhealthy Days: <strong>{station.unhealthyDays}</strong></span>
							</div>
						</div>
					</div>
				</div>
			{/each}
		</div>

		<!-- Right side: legend -->
		<div class="legend-section">
			<h4>Air Quality Index Levels</h4>
			{#each aqiCategories as cat}
				<div class="legend-item-new">
					<div class="legend-color" style="background-color: {cat.color}"></div>
					<div class="legend-text">
						<div class="legend-name">{cat.name}</div>
						<div class="legend-range">{cat.min}–{cat.max}</div>
					</div>
				</div>
			{/each}
		</div>
	</div>

	<!-- COMPARISON VIEW -->
	<!-- Detailed side-by-side comparison when stations are selected -->
	{#if selectedStationData.length > 0}
		<div class="comparison-box">
			<h4>
				Detailed Comparison
				{#if selectedStationData.length === 1}
					(Select another station to compare)
				{/if}
			</h4>

			<div class="comparison-grid" class:single={selectedStationData.length === 1}>
				{#each selectedStationData as station}
					<!-- Categorize stations based on percentage of good air days -->
					{@const recClass =
						station.goodPercent > 60
							? 'excellent'
							: station.goodPercent > 40
								? 'acceptable'
								: 'poor'}
					{@const recText =
						station.goodPercent > 60
							? 'Excellent choice for people with respiratory issues'
							: station.goodPercent > 40
								? 'Acceptable, but monitor air quality regularly'
								: 'Consider other neighborhoods if you have asthma'}

					<div class="comparison-card">
						<h4 class="comparison-title">{station.station}</h4>

						<div class="stat-grid">
							<div class="stat-item">
								<div>Average AQI</div>
								<div class="stat-value" style="color: {getCategory(station.avgAqi).color}">
									{station.avgAqi.toFixed(0)}
								</div>
							</div>

							<div class="stat-item">
								<div>Median AQI</div>
								<div class="stat-value" style="color: {getCategory(station.medianAqi).color}">
									{station.medianAqi.toFixed(0)}
								</div>
							</div>

							<div class="stat-item">
								<div>Worst Day</div>
								<div class="stat-value" style="color: #f65e5f">
									{station.maxAqi.toFixed(0)}
								</div>
							</div>

							<div class="stat-item">
								<div>Total Days</div>
								<div class="stat-value">
									{station.totalDays}
								</div>
							</div>
						</div>

						<div class="breakdown-box">
							<div class="breakdown-item">
								<span style="color: #9cd84e; font-weight: bold">Good:</span>
								{station.goodDays} days ({station.goodPercent.toFixed(1)}%)
							</div>
							<div class="breakdown-item">
								<span style="color: #d4a017; font-weight: bold">Moderate:</span>
								{station.moderateDays} days ({station.moderatePercent.toFixed(1)}%)
							</div>
							<div class="breakdown-item">
								<span style="color: #f65e5f; font-weight: bold">Unhealthy:</span>
								{station.unhealthyDays} days ({station.unhealthyPercent.toFixed(1)}%)
							</div>
						</div>

						<div class="recommendation {recClass}">
							{recText}
						</div>
					</div>
				{/each}
			</div>

			<!-- Head-to-head comparison when exactly 2 stations are selected -->
			{#if selectedStationData.length === 2}
				{@const better =
					selectedStationData[0].goodPercent > selectedStationData[1].goodPercent
						? selectedStationData[0]
						: selectedStationData[1]}
				{@const worse =
					selectedStationData[0].goodPercent > selectedStationData[1].goodPercent
						? selectedStationData[1]
						: selectedStationData[0]}

				<div class="head-to-head">
					<strong>Head-to-Head:</strong>
					<span style="color: #9cd84e; font-weight: bold">{better.station}</span> has
					{(better.goodPercent - worse.goodPercent).toFixed(1)}% more good air days than
					<strong>{worse.station}</strong>
				</div>
			{/if}
		</div>
	{/if}
</div>

<style>
	/* Global font styling */
	* {
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
	}

	p {
		font-size: 14px;
		display: flex;
		align-items: center;
	}

	.container {
		max-width: 1040px;
	}

	.ranking-item {
		padding: 15px;
		background: white;
		border: 1px solid #e0e0e0;
		border-radius: 8px;
		margin-bottom: 10px;
	}

	.ranking-item:hover {
		background: #f5f5f5;
	}

	.ranking-item.selected {
		background: #fff3cd;
		border: 1px solid #ffc107;
	}

	.ranking-content {
		display: flex;
		align-items: center;
		gap: 15px;
	}

	.rank-number {
		font-size: 24px;
		font-weight: bold;
		min-width: 40px;
		color: #666;
	}

	.station-info {
		flex: 1;
	}

	.station-name {
		font-size: 16px;
		font-weight: bold;
		margin-bottom: 8px;
	}

	.bar-container {
		display: flex;
		height: 30px;
		border-radius: 8px;
		overflow: hidden;
	}

	.bar-segment {
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 13px;
		font-weight: bold;
	}

	.bar-good {
		background: #9cd84e80;
		color: black;
	}

	.bar-moderate {
		background: #facf3980;
		color: #000;
	}

	.bar-unhealthy {
		background: #f9904980;
		color: black;
	}

	.station-stats {
		font-size: 13px;
		color: #666;
		margin-top: 8px;
		display: flex;
		gap: 15px;
	}

	.comparison-box {
		background: #f8f9fa;
		padding: 25px;
		padding-top: 0px;
		border-radius: 8px;
		border: 1px solid #e0e0e0;
		margin-top: 30px;
	}

	.comparison-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 20px;
	}

	.comparison-grid.single {
		grid-template-columns: 1fr;
	}

	.comparison-card {
		background: white;
		padding: 20px;
		border-radius: 8px;
		border: 1px solid #ddd;
	}

	.comparison-title {
		margin-top: 0;
		font-size: 16px;
		color: black;
	}

	.stat-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 15px;
		margin-bottom: 15px;
	}

	.stat-item {
		font-size: 13px;
		color: #666;
	}

	.stat-value {
		font-size: 28px;
		font-weight: bold;
	}

	.breakdown-box {
		padding: 15px;
		background: #f8f9fa;
		border-radius: 8px;
		font-size: 14px;
		border: 1px solid #ddd;
	}

	.breakdown-item {
		margin-bottom: 8px;
	}

	.breakdown-item:last-child {
		margin-bottom: 0;
	}

	.recommendation {
		margin-top: 15px;
		padding: 12px;
		border-radius: 8px;
		font-size: 14px;
		font-weight: bold;
	}

	.recommendation.excellent {
		background: #d4edda;
		border: 1px solid #28a745;
	}

	.recommendation.acceptable {
		background: #fff3cd;
		border: 1px solid #ffc107;
	}

	.recommendation.poor {
		background: #f8d7da;
		border: 1px solid #dc3545;
	}

	.head-to-head {
		margin-top: 20px;
	}

	.rankings-wrapper {
		display: flex;
		gap: 30px;
		align-items: flex-start;
	}

	.rankings-list {
		flex: 1;
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

	.legend-item-new {
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

	.legend-name {
		color: #333;
		line-height: 1.1;
	}

	.legend-range {
		font-size: 10px;
		color: #666;
	}

	.legend-text {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}
</style>
