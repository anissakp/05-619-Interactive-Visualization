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

    // CONSTANTS AND CONFIGURATION


    // DERIVED DATA


    // SCALES

	// Your new chart code here...
</script>

<!-- CONTROLS -->


<!-- LEGENDS -->



<svg width={800} height={400}>
	<!-- Your visualization -->
</svg>