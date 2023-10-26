<script>
  import { onMount, afterUpdate } from 'svelte';
  import { dateStore } from './DateStore.js'; // Import the store
import { Chart, ColumnSeries, LineSeries, Category, Legend, Tooltip, SplineSeries, BarSeries, DateTime, } from '@syncfusion/ej2-charts';
import { Browser } from '@syncfusion/ej2-base';
Chart.Inject(ColumnSeries, LineSeries, Category, Legend, Tooltip, SplineSeries,BarSeries, DateTime);
import Card from './MetricsCard.svelte';
import { format, parse, compareAsc } from 'date-fns';

export let appData;

let chartDataStateReasons = [];
  let startDate = '';
  let endDate = '';
  let previousStartDate = '';
  let previousEndDate = '';

  // Function to check if local storage has date information
  function getDatesFromLocalStorage() {
    const storedStartDate = localStorage.getItem('startDate');
    const storedEndDate = localStorage.getItem('endDate');

    if (storedStartDate && storedEndDate) {
      startDate = storedStartDate;
      endDate = storedEndDate;
    }
  }

  // Subscribe to the dateStore
  dateStore.subscribe((value) => {
    previousStartDate = startDate; // Store the previous start date
    previousEndDate = endDate; // Store the previous end date
    startDate = value.startDate;
    endDate = value.endDate;
   // Fetch data whenever the date changes
       // Check if the dates have changed before making an API call
       if (startDate !== previousStartDate || endDate !== previousEndDate) {
        fetchOpportunityStateReasonsChartData(startDate, endDate); 
      // Store the dates in local storage for future use
      localStorage.setItem('startDate', startDate);
      localStorage.setItem('endDate', endDate);
    }
  });



  onMount(() => {
    getDatesFromLocalStorage(); // Try to get dates from local storage
   // Fetch data on component mount
  });

  afterUpdate(() => {
    fetchOpportunityStateReasonsChartData(startDate, endDate); // Fetch data after updates
    // Store the dates in local storage for future use
    localStorage.setItem('startDate', startDate);
    localStorage.setItem('endDate', endDate);
  });
async function fetchOpportunityStateReasonsChartData(startDate, endDate) {
  // Use selectedStartDate and selectedEndDate in the API call
  const apiUrlStateReasons = `${appData.service.endpoint}/dashboard/sales/data/opportunity/statereasons?start=${startDate}&end=${endDate}&apiKey=${appData.service.apiKey}`;
  const responseStateReasons = await fetch(apiUrlStateReasons);
  const dataStateReasons = await responseStateReasons.json();
  // Process the API response data for the Opportunity State Reasons chart
  const chartDataStateReasons = dataStateReasons.map(item => ({
    x: item.stateReason,
    y: item.count
  }));

  // Check if chartDataStateReasons is empty
  if (chartDataStateReasons.length === 0) {
      const noDataContainer = document.getElementById('chart-no-data');
      noDataContainer.style.display = 'block';
    } else {
      const noDataContainer = document.getElementById('chart-no-data');
      noDataContainer.style.display = 'none';

  console.log(chartDataStateReasons);
  const chartStateReasons = new Chart({
    primaryXAxis: {
      valueType: 'Category',
      majorGridLines: { width: 0 }
    },
    primaryYAxis: {
      labelFormat: '{value}',
      edgeLabelPlacement: 'Shift',
      majorTickLines: { width: 0 },
      lineStyle: { width: 0 },
    },
    series: [
      {
        type: 'Bar',
        dataSource: chartDataStateReasons,
        xName: 'x',
        width: 2,
        yName: 'y',
        name: 'Count',
        fill: 'Dodgerblue',
        columnSpacing: 0.1,
      },
    ],
  });

  chartStateReasons.appendTo('#chart-container-state-reasons');
};
}
</script>

<div class="chart-card">
  <h2>Opportunity State Reasons</h2>
  <div id='chart-no-data'>No data found.</div>
  <div id='chart-container-state-reasons'></div>
</div>

<style>
    #chart-no-data {
    text-align: center;
    display: none;
    font-weight: bold;
    font-size: 16px;
    color: gray;
  }

  h2 {
    font-weight: bold;
    font-size: large;
    margin-left: 20px;
    margin-top: 20px;
  }
  
  .chart-card {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    margin: 0px; /* Add margin to create space between the charts */
    margin-top: 20px;
  }
</style>
