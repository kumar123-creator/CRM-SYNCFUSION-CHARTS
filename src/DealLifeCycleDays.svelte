<script>
  import { onMount, afterUpdate } from 'svelte';
  import { dateStore } from './DateStore.js'; // Import the store
import { DateRangePicker } from '@syncfusion/ej2-calendars';
import { Chart, ColumnSeries, LineSeries, Category, Legend, Tooltip, SplineSeries, BarSeries, DateTime } from '@syncfusion/ej2-charts';
import { Browser } from '@syncfusion/ej2-base';
Chart.Inject(ColumnSeries, LineSeries, Category, Legend, Tooltip, SplineSeries,BarSeries, DateTime);
import Card from './MetricsCard.svelte';
import { format, parse, compareAsc } from 'date-fns';


    const currentDate = new Date();
     const currentYear = currentDate.getFullYear();
     export let appData;
     let startDate = '';
     let endDate = '';
 
     // Calculate the start and end date objects
     const startDateObj = new Date(startDate);
     const endDateObj = new Date(endDate);
 
     // Calculate the range of months between the start and end date
     const startYear = startDateObj.getFullYear();
     const endYear = endDateObj.getFullYear();
     const startMonth = startDateObj.getMonth();
     const endMonth = endDateObj.getMonth();
 
     const monthNames = [];
 
     for (let year = startYear; year <= endYear; year++) {
         const start = (year === startYear) ? startMonth : 0;
         const end = (year === endYear) ? endMonth : 11;
 
         for (let month = start; month <= end; month++) {
             monthNames.push(`${year} ${new Date(year, month, 1).toLocaleString('default', { month: 'short' })}`);
         }
     }
 
 
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
     fetchOpportunityChartData(startDate, endDate); // Fetch data whenever the date changes
   });

   onMount(() => {
     getDatesFromLocalStorage(); // Try to get dates from local storage
     fetchOpportunityChartData(startDate, endDate); // Fetch data on component mount
   });
 
   afterUpdate(() => {
     fetchOpportunityChartData(startDate, endDate); // Fetch data after updates
     // Store the dates in local storage for future use
     localStorage.setItem('startDate', startDate);
     localStorage.setItem('endDate', endDate);
   });
async function fetchOpportunityChartData(startDate, endDate) {
    // Use selectedStartDate and selectedEndDate in the API call
    const apiUrlDays = `${appData.service.endpoint}/dashboard/sales/data/opportunitymonthlymetrics?start=${startDate}&end=${endDate}&apiKey=${appData.service.apiKey}`;
    const responseDays = await fetch(apiUrlDays);
    const dataDays = await responseDays.json();


    const data = dataDays.map(item => {
  const month = parseInt(item.monthLabel.split('/')[0]) - 1;
  const year = parseInt(item.monthLabel.split('/')[1]);
  const monthLabel = new Date(year, month, 1);
  const formattedDate = new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'short'
  }).format(monthLabel);

  return {
    x: formattedDate, // Format as "Jan 2023" (month name and year)
    opportunities: item.opportunities,
    days: item.days
  };
});

// Sort the data by the 'x' property (which represents formatted month and year)
data.sort((a, b) => {
  const dateA = new Date('01 ' + a.x);
  const dateB = new Date('01 ' + b.x);
  return dateA - dateB;
});


      console.log(data);
	  
	  const chartDays = new Chart({
		primaryXAxis: {
		  valueType: 'Category',
		  majorGridLines: { width: 0 }
		},
		primaryYAxis: {
		  labelFormat: '{value}',
		  title: 'Opportunities',
		  edgeLabelPlacement: 'Shift',
		  majorTickLines: { width: 0 },
		  lineStyle: { width: 0 },
		  majorGridLines: { width: 1 },
		},
		series: [
		  {
			type: 'Column',
			dataSource: data,
			xName: 'x',
			yName: 'opportunities',
			name: 'Opportunities',
      fill: 'DodgerBlue',
			columnSpacing: 0.1,
		  },
		  {
			type: 'Spline',
			dataSource: data,
			xName: 'x',
			yName: 'days',
			name: 'Avg Days to Deal',
			yAxisName: 'rightYAxis',
      fill: 'Purple',
			marker: {
			  visible: true,
			  height: 10,
			  width: 10,
			},
		  }
		],
		axes: [
		  {
			name: 'rightYAxis',
			opposedPosition: true,
			title: 'Total Time to Deal Days',
			majorGridLines: { width: 1 },
		  },
		],
		legendSettings: {
		  visible: true,
		},
		tooltip: {
       enable: true, 
      format: '${point.x}: ${point.y}' },
		width: '100%',
		height: '300px'
	  });
  
	  chartDays.appendTo('#chart-container-days');
  }
</script>


<div class="chart-card">
  <h2>Deal Lifecycle Days</h2>
  <div id='chart-container-days'></div>
</div>
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
