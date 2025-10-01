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

	// provided code for AQI levels
	[
		{ name: 'Good', min: 0, max: 50, color: '#9cd84e' },
		{ name: 'Moderate', min: 51, max: 100, color: '#facf39' },
		{ name: 'Unhealthy for Sensitive Groups', min: 101, max: 150, color: '#f99049' },
		{ name: 'Unhealthy', min: 151, max: 200, color: '#f65e5f' },
		{ name: 'Very Unhealthy', min: 201, max: 300, color: '#a070b6' },
		{ name: 'Hazardous', min: 301, color: '#a06a7b' }
	];

	// just for debugging; can be removed
	$inspect(data);
</script>

<pre>
{JSON.stringify(data[0], null, 2)}
</pre>

<style>
	svg {
		font-family: sans-serif;
	}
</style>
