<script lang="ts">
	import AQIChart from '$lib/AQIChart.svelte';
	import RankCompareChart from '$lib/RankCompareChart.svelte';
	import * as d3 from 'd3';

	// DATA SOURCES

	// URLs for all station station CSV files
	const datasets = {
		avalon: 'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BAvalon%5D_daily-avg.csv',
		glassport_high_street:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BGlassport%20High%20Street%5D_daily-avg.csv',
		lawrenceville:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BLawrenceville%5D_daily-avg.csv',
		liberty_sahs:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BLiberty%20(SAHS)%5D_daily-avg.csv',
		manchester:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BManchester%5D_daily-avg.csv',
		north_braddock:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BNorth%20Braddock%5D_daily-avg.csv',
		parkway_east_near_road:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BParkway%20East%20(Near%20Road)%5D_daily-avg.csv',
		usa_pennsylvania_pittsburgh:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BUSA-Pennsylvania-Pittsburgh%5D_daily-avg.csv'
	};

	// DATA LOADING

	// Load all CSV files in parallel and combine into single dataset
	// Note: USA-Pennsylvania-Pittsburgh CVS lacks "Station Name" column, so a fallback value is provided
	const data = Promise.all(
		Object.values(datasets).map((url) =>
			d3.csv(url, (d: any) => ({
				city: d.City,
				country: d.Country,
				mainPollutant: d['Main pollutant'],
				pm25: +d['PM2.5'],
				state: d.State,
				stationName: d['Station name'] || 'USA-Pennsylvania-Pittsburgh',
				timestamp: new Date(d['Timestamp(UTC)']),
				usAqi: +d['US AQI']
			}))
		)
	).then((allData) => allData.flat());
</script>

{#await data}
	<!-- promise is pending -->
	<p>loading data...</p>
{:then data}
	<!-- promise was fulfilled or not a Promise -->
	<h2>Monthly Air Quality Trends by Station</h2>
	<AQIChart {data} />

	<br>
	<br>

	<h2>Air Quality Rankings and Comparison</h2>              
	<RankCompareChart {data} />                   

{:catch error}
	<!-- promise was rejected -->
	<p>Something went wrong: {error.message}</p>
{/await}

<style>
	* {
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
	}
</style>
