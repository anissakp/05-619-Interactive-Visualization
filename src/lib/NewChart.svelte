<!-- src/lib/NewChart.svelte -->
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
		{ name: 'Unhealthy for Sensitive Groups', min: 101, max: 150, color: '#f99049' },
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
	<!-- INTRO BOX -->
	<!-- svelte-ignore a11y_label_has_associated_control -->
	<p>
		Goal: Find the best neighborhood for people with asthma or respiratory issues.
	</p>
	<p>
		How to use: Click on any neighborhood bar to see details. Click two neighborhoods
		to compare side-by-side!
	</p>

	<!-- RANKINGS -->
	<h4>Rankings: Click to Select (up to 2 for comparison)</h4>

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
					<div class="station-stats">
						<span>Avg AQI: <strong>{station.avgAqi.toFixed(0)}</strong></span>
						<span>Good Days: <strong>{station.goodDays}</strong></span>
						<span>Unhealthy Days: <strong>{station.unhealthyDays}</strong></span>
					</div>
				</div>
			</div>
		</div>
	{/each}

	<!-- COMPARISON VIEW -->
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

	<!-- LEGEND -->
	<h4 style="margin-top: 30px">What the Colors Mean</h4>
	<div class="legend-grid">
		{#each aqiCategories as cat}
			<div class="legend-item" style="background: {cat.color}30;">
				<div style="font-size: 16px; font-weight: bold; margin-bottom: 5px">
					{cat.name}
				</div>
				<div style="font-size: 14px; color: #666">
					{cat.min}-{cat.max ?? '301+'} AQI
				</div>
			</div>
		{/each}
	</div>
</div>

<style>
	* {
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif
	}

	p {
		font-size: 14px;
		display: flex;
		align-items: center;
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

	.legend-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
		gap: 15px;
		margin-top: 15px;
	}

	.legend-item {
		padding: 15px;
		border-radius: 8px;
	}
</style>