# Pittsburgh Air Quality Visualizations

This project was completed as part of Carnegie Mellon University’s 05-619: Data Visualization course. Its goal was to gain familiarity with implementing interactive visualization techniques using **Svelte and D3**, while evaluating how effective these techniques are within a chosen data domain. In this case, air quality data across Pittsburgh-area monitoring stations.

[**Deployed Page**](https://anissakp.github.io/Pittsburgh-Air-Quality-Visualizations/)
 

## Installation and Usage
Follow these steps to set up the project locally:

1. **Clone the Repository**
```bash
git clone https://github.com/anissakp/Pittsburgh-Air-Quality-Visualizations.git
```

2. **Install Dependencies at Root Directory**
```bash
cd Pittsburgh-Air-Quality-Visualizations
npm install
```

3. **Start Development Server**
```bash
npm run dev
```

## My Process
The project began by reviewing previous lab assignments and examples from Professor Moritz’s GitHub repository (e.g., FullSVGBarChart.svelte), which helped develop an understanding of Svelte and D3. Using these tools, I developed **interactive visualizations** that display how air quality differs across Pittsburgh’s neighborhoods and highlights areas with the cleanest air.

### Visualization 1: AQIChart.svelte
This visualization lets users explore how air quality changes over time across Pittsburgh’s neighborhoods. Monthly average AQI values are shown as a line chart, with D3’s categorical color scale distinguishing each neighborhood. Background color bands represent EPA-defined AQI levels (Good → Hazardous) at 50% opacity for easy reference. When a single station is selected, a shaded 10th-90th percentile band highlights daily variability.

Interactive features include a dropdown to choose stations, a checkbox to toggle raw daily data, and a fixed legend for clarity. Together, these elements help users move smoothly between overview and detail without losing context.

This visualization took approximately **15-20 hours** to complete. 

### Visualization 2: RankCompareChart.svelte
Initially, I explored a map-based visualization to show how air quality changes across Pittsburgh over time. My plan was to plot circles on a city map, sized by PM2.5 values, colored by AQI level, with tooltips on hover. However, the cities in the dataset weren’t spread out enough for the map to be clear or effective. I also considered an interactive dashboard showing all eight counties to make it easier to compare stations, but this approach was difficult to read and didn’t clearly answer a specific question.

This prompted a shift toward the question: **Which Pittsburgh neighborhoods have the best air quality for individuals with respiratory conditions such as asthma?** To answer this, stations are ranked by the percentage of “good air” days (a metric closely tied to daily quality of life). Each neighborhood is shown as a stacked horizontal bar representing the proportion of days in each AQI category, ordered by rank and colored consistently with the time-series chart.

Users can compare up to two stations side by side, with a panel displaying average and median AQI, the worst recorded day, total data days, category breakdowns, and qualitative ratings (excellent/acceptable/poor).

These design decisions were guided by peer feedback and aimed to make the data easier to interpret at a glance, show meaningful differences between neighborhoods, and support both broad exploration and focused comparison.

This process took approximately **20-25 hours** to complete.

### Most Time-Consuming Aspects
The most time-consuming part of the project was learning how to work with Svelte and D3. Both have extensive documentation and multiple ways to accomplish the same task, so a lot of effort went into figuring out the most concise and reliable approach. Styling also took time, as I spent a lot of trial and error making the charts, legends, and tooltips visually consistent and easy to interpret.

### Use of AI
I used **Claude by Anthropic** for clarifying confusing documentation, providing code examples based on documentation, troubleshooting D3-Svelte integration issues, resolving TypeScript errors, and refining HTML/CSS styling. A list of prompts can be found here. 
