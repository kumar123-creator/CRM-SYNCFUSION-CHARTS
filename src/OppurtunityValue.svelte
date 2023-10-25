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
      startDate = value.startDate;
      endDate = value.endDate;
      fetchOpportunityValueByUserChartData(startDate, endDate); // Fetch data whenever the date changes
  });

  onMount(() => {
      getDatesFromLocalStorage();
      fetchOpportunityValueByUserChartData(startDate, endDate);
  });

  afterUpdate(() => {
      fetchOpportunityValueByUserChartData(startDate, endDate);
      updateChart();
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

              const monthNames = [...new Set(jsonData.map((item) => item.monthLabel))];

                        // Sort the month names in ascending order
                        monthNames.sort((a, b) => {
                            const [aMonth, aYear] = a.split('/');
                            const [bMonth, bYear] = b.split('/');
                            if (+aYear !== +bYear) {
                                return +aYear - +bYear;
                            } else {
                                return +aMonth - +bMonth;
                            }
                        });

              // Dynamically extract user names from the data
              userNames = Object.keys(jsonData[0]).filter((key) => key !== 'monthLabel');

              // Transform API data into the format suitable for the chart
              chartData = monthNames.map((monthName) => {
                  const monthData = jsonData.filter((item) => item.monthLabel === monthName);

                  const transformedData = { x: monthName };
                  userNames.forEach((userName) => {
                      transformedData[userName] = monthData.reduce((total, item) => total + item[userName], 0) / 1;
                  });

                  return transformedData;
              });

              // After fetching data, update the chart
              updateChart();
          } else {
              console.error('Failed to fetch data from the API. HTTP status:', response.status);
          }
      } catch (error) {
          console.error('An error occurred while fetching data:', error);
      }
  }

  const colors = [ 'DodgerBlue', 'Tomato', 'Gold', 'LimeGreen', 'Purple', 'Orange', 'Crimson', 'RoyalBlue'];


  // Function to create or update the chart

  function updateChart() {
      let lastYear = null;
      chartData = chartData.map((item, index) => {
  const dateParts = item.x.split('/');
  const month = parseInt(dateParts[0]);
  const year = parseInt(dateParts[1]);

  const monthAbbreviation = new Date(year, month - 1, 1).toLocaleString('default', { month: 'short' });

  // Display both the year and month
  const formattedDate = `${monthAbbreviation} ${year}`;

  return {
      x: formattedDate,
      ...userNames.reduce((acc, userName) => {
          acc[userName] = item[userName];
          return acc;
      }, {}),
  };
});
        // Create and append the chart with the updated data source
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
                title: 'Oppurtunity value',
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
        format: '${point.x}: GBP ${point.y}',
      },
            legendSettings: { enableHighlight: true },
        });
        chart.appendTo('#chart-container-opportunity');
  }




</script>

<style>

.center-container {
    display: flex;
    justify-content: flex-end;
    align-items: flex-end;
    margin-top: 60px;
    position: absolute;
    top: 0;
    right: 0;
  }

  .chart-card {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    margin: 10px; /* Add margin to create space between the charts */
  }

	main {
	  display: flex;
	  flex-wrap: wrap;
	  justify-content: center;
    margin-top: 100px;
	}
  
  .card {
    flex: 1;
    max-width: 300px;
    margin: 5px;
  }

  /* Tooltip container style */
  .tooltip {
    position: relative;
    display: inline-block;
  }

  .tooltiptext {
    visibility: hidden;
    width: 200px;
    background-color: #333;
    color: #fff;
    text-align: center;
    border-radius: 4px;
    padding: 5px;
    position: absolute;
    z-index: 1;
    bottom: 125%; /* Position the tooltip above the card */
    left: 50%;
    transform: translateX(-50%);
    opacity: 0;
    transition: opacity 0.2s;
  }

  .tooltip:hover .tooltiptext {
    visibility: visible;
    opacity: 1;
  }
  h2 {
    font-weight: bold;
    font-size: large;
  }
</style>

<div class="chart-card">
  <h2>Opportunity Value by User</h2>
  <div id='chart-container-opportunity'></div>
</div> 
