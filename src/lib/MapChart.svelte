<!-- TO-DO: Create an interactive map visualization of air quality data across Pennsylvania -->
<!-- ✅ Set up chart dimensions and margins -->
<!-- ✅ Load GeoJSON data for Pennsylvania map -->
<!-- ✅ Add station coordinates lookup table -->
<!-- ✅ Create geographic projection for Pennsylvania -->
<!-- Create color scale for AQI levels -->
<!-- Create size scale for PM2.5 values -->
<!-- Render the base Pennsylvania map -->
<!-- Plot station circles with color (AQI) and size (PM2.5) -->
<!-- Add time slider to filter by month/week -->
<!-- Add play/pause button for time animation -->
<!-- Add city dropdown filter -->
<!-- Add pollutant dropdown filter -->
<!-- Add AQI range slider filter -->
<!-- Add hover tooltip showing station details -->
<!-- Add hover effects (enlarge, outline, pulse) -->
<!-- Add AQI color legend -->
<!-- Add PM2.5 size legend -->
<!-- Final styling and polish -->

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

    // GeoJSON type definition
	// Source: GeoJSON specification
	interface GeoJSONData {
		type: string;
		features: Array<{
			type: string;
			properties: any;
			geometry: any;
		}>;
	}

    // PROPS
	const { data }: { data: Item[] } = $props();

    // STATE VARIABLES


    // CONSTANTS AND CONFIGURATION
    // Map Dimensions
	const width = 900;
	const height = 600;
	const margin = { top: 20, right: 20, bottom: 80, left: 20 };

    // Calculate the actual drawing area (excluding margins)
	const mapWidth = width - margin.left - margin.right;
	const mapHeight = height - margin.top - margin.bottom;

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

    // Station coordinates lookup table [longitude, latitude]
    // Coordinates are [longitude, latitude] for D3 projection
	// Source: Google search for each city
	const stationCoordinates: Record<string, [number, number]> = {
		'Avalon': [-80.0695, 40.5009],
		'Glassport High Street': [-79.8892, 40.3256],
		'Lawrenceville': [-79.9606, 40.4668],
		'Liberty (SAHS)': [-79.8878, 40.3231],
		'Manchester': [-80.0092, 40.4562],
		'North Braddock': [-79.8542, 40.3967],
		'Parkway East (Near Road)': [-79.8672, 40.4378],
		'USA-Pennsylvania-Pittsburgh': [-79.9959, 40.4406]
	};


    // DATA LOADING
	// Load Pennsylvania GeoJSON data
	// Source: https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json
	const geoData = d3.json('https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json') as Promise<GeoJSONData>;


    //GEOGRAPHIC PROJECTION
	// Create a geographic projection to convert lat/lon to x/y coordinates
	// Source: https://d3js.org/d3-geo/conic#geoAlbers
	// Pennsylvania center approximately: [-77.5, 40.9]
	const projection = d3.geoAlbers()
		.center([-77.5, 40.9])            // Center on Pennsylvania
		.scale(6000)                      // Zoom level (adjust to fit Pennsylvania)
		.translate([mapWidth / 2, mapHeight / 2]);  // Center in drawing area
	
	// Path generator converts GeoJSON geometries to SVG path strings
	// Source: https://d3js.org/d3-geo/path
	const pathGenerator = d3.geoPath().projection(projection);



    // SCALES

	// Your new chart code here...
</script>

<!-- CONTROLS -->

{#await geoData}
	<p>Loading map data...</p>
{:then geo}
	<div>Map loaded! GeoJSON has {geo.features.length} features</div>
{:catch error}
	<p>Error loading map: {error.message}</p>
{/await}


<!-- LEGENDS -->



<svg width={800} height={400}>
	<!-- Your visualization -->
</svg>