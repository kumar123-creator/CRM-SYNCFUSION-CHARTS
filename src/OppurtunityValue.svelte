<script>
  import { onMount, afterUpdate } from 'svelte';
  import { Chart, StackingColumnSeries, Category, DataLabel, Tooltip, Legend,Highlight } from '@syncfusion/ej2-charts';
  import { Browser } from '@syncfusion/ej2-base';
  import { dateStore } from './DateStore.js'; // Import the store
  Chart.Inject(StackingColumnSeries, Category, DataLabel, Tooltip, Legend, Highlight);

  export let appData;
  let chartData = []; // Initialize with an empty array
  let startDate = '';
  let endDate = '';
  let userNames = []; // Array to hold user names
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
      fetchOpportunityValueByUserChartData(startDate, endDate);
      // Store the dates in local storage for future use
      localStorage.setItem('startDate', startDate);
      localStorage.setItem('endDate', endDate);
    }
  });

  onMount(() => {
      getDatesFromLocalStorage();
      
  });

  afterUpdate(() => {
    updateChart();
      fetchOpportunityValueByUserChartData(startDate, endDate);
    
      localStorage.setItem('startDate', startDate);
      localStorage.setItem('endDate', endDate);
  });
  async function fetchOpportunityValueByUserChartData(startDate, endDate) {
      try {
          const response = await fetch(
            `${appData.service.endpoint}/dashboard/sales/data/opportunitymonthlyusermetrics?start=${startDate}&end=${endDate}&apiKey=${appData.service.apiKey}`
          );
          if (response.ok) {
              const jsonData = await response.json();
              console.log(jsonData);

              if (jsonData.length === 0) {
                  chartData = [];
              } else {
                  const monthNames = [...new Set(jsonData.map((item) => item.monthLabel))];
                  monthNames.sort((a, b) => {
                      const [aMonth, aYear] = a.split('/');
                      const [bMonth, bYear] = b.split('/');
                      if (+aYear !== +bYear) {
                          return +aYear - +bYear;
                      } else {
                          return +aMonth - +bMonth;
                      }
                  });

                  userNames = Object.keys(jsonData[0]).filter((key) => key !== 'monthLabel');

                  chartData = monthNames.map((monthName) => {
                      const monthData = jsonData.filter((item) => item.monthLabel === monthName);
                      const transformedData = { x: monthName };
                      userNames.forEach((userName) => {
                          transformedData[userName] = monthData.reduce((total, item) => total + item[userName], 0) / 1;
                      });
                      return transformedData;
                  });
              }

              updateChart();
          } else {
              console.error('Failed to fetch data from the API. HTTP status:', response.status);
          }
      } catch (error) {
          console.error('An error occurred while fetching data:', error);
      }
  }

  const colors = ['DodgerBlue', 'Tomato', 'Gold', 'LimeGreen', 'Purple', 'Orange', 'Crimson', 'RoyalBlue'];

  function updateChart() {
      let lastYear = null;
      chartData = chartData.map((item, index) => {
          const dateParts = item.x.split('/');
          const month = parseInt(dateParts[0]);
          const year = parseInt(dateParts[1]);
          const monthAbbreviation = new Date(year, month - 1, 1).toLocaleString('default', { month: 'short' });
          const formattedDate = `${monthAbbreviation} ${year}`;
          return {
              x: formattedDate,
              ...userNames.reduce((acc, userName) => {
                  acc[userName] = item[userName];
                  return acc;
              }, {}),
          };
      });

      const chart = new Chart({
          primaryXAxis: {
              valueType: 'Category',
              majorGridLines: { width: 0 },
              labelStyle: {
                  size: '15px',
                  fontWeight: 'normal',
              },
          },
          primaryYAxis: {
              edgeLabelPlacement: 'Shift',
              majorTickLines: { width: 0 },
              minorTickLines: { width: 0 },
              lineStyle: { width: 0 },
              labelFormat: '{value}',
              labelPlacement: 'OnTicks',
              title: 'Opportunity value',
              labelStyle: {
                  size: '15px',
                  fontWeight: 'normal',
              },
          },
          height: '350px',
          series: userNames.map((userName, index) => ({
              type: 'StackingColumn',
              dataSource: chartData,
              xName: 'x',
              width: 2,
              fill: colors[index],
              yName: userName,
              columnSpacing: 0.2,
              name: userName,
              columnWidth: 0.5,
          })),
          tooltip: {
              enable: true,
              format: '${point.x}: ${point.y} GBP',
          },
          legendSettings: { enableHighlight: true },
      });
      chart.appendTo('#chart-container-opportunity');
  }



</script>

<div class="chart-card">
  <h2>Opportunity Value by User</h2>
  <div id='chart-container-opportunity'></div>
</div> 

<style>
  .chart-card {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    margin: 0px; /* Add margin to create space between the charts */
    margin-top: 20px;
  }
  h2 {
    font-weight: bold;
    font-size: large;
  }
</style>
