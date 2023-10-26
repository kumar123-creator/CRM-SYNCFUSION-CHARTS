<script>
 import Card from './Card.svelte';
  import { onMount,afterUpdate } from 'svelte';
  import { dateStore } from './DateStore.js'; // Import the store
  

 export let appData;
   let data = null; 
   let startDate = "";
   let endDate = "";
   let previousStartDate = '';
  let previousEndDate = '';

   function getDatesFromLocalStorage() {
  const storedStartDate = localStorage.getItem('startDate');
  const storedEndDate = localStorage.getItem('endDate');

  if (storedStartDate && storedEndDate) {
    startDate = storedStartDate;
    endDate = storedEndDate;
    // Fetch data when dates are loaded from local storage

  }
}

    // Subscribe to the dateStore
    dateStore.subscribe((value) => {
  startDate = value.startDate;
  endDate = value.endDate;
    // Check if the dates have changed before making an API call
    if (startDate !== previousStartDate || endDate !== previousEndDate) {
      fetchData(startDate, endDate);
      // Store the dates in local storage for future use
      localStorage.setItem('startDate', startDate);
      localStorage.setItem('endDate', endDate);
    }
});


async function fetchData(startDate, endDate) {

try {
  // Use startDate and endDate in your API calls
  const response = await fetch(`${appData.service.endpoint}/dashboard/sales/metrics?start=${startDate}&end=${endDate}&apiKey=${appData.service.apiKey}`,
    {
      headers: {
        'Cookie': 'SESSION=NTkwN2VlOWQtZjRlNi00NmQ4LWE4MTUtOTJhNT71YjA0ZWMx',
      },
    }
  );
  data = await response.json();
} catch (error) {
  console.error('Error fetching data:', error);
}
}
console.log(data);
    onMount(() => {
      getDatesFromLocalStorage(); // Try to get dates from local storage
      fetchData(startDate, endDate); // Fetch data on component mount
    });
  
    afterUpdate(() => {
   
      localStorage.setItem('startDate', startDate);
      localStorage.setItem('endDate', endDate);
    });
 </script>
 <main>
 {#if data}
 <div class="card tooltip">
   <Card title="Leads Converted" value={data.leadsConverted} icon="fal fa-lg fa-check-square" />
   <span class="tooltiptext">Leads Converted</span>
 </div>

 <div class="card tooltip">
   <Card title="Opportunities" value={data.opportunityCountOpen} icon="fal fa-lg fa-fire" />
   <span class="tooltiptext">Opportunities</span>
 </div>

 <div class="card tooltip">
   <Card title={`Won from ${data.opportunityCountWon} deals`} value={`GBP ${data.opportunityValueWon}`} icon="fas fa-dollar-sign" />
   <span class="tooltiptext">Oppurtunity Deal Value</span>
 </div>

 <div class="card tooltip">
   <Card title="Avg. Deal Cycle" value={data.avgDealCycleDays} icon="fal fa-lg fa-stopwatch" />
   <span class="tooltiptext">Avg. Deal Cycle Days</span>
 </div>

 <div class="card tooltip">
   <Card title="Avg. Deal Value" value={`GBP ${data.avgDealValue}`} icon="fas fa-dollar-sign" />
   <span class="tooltiptext">Avg. Deal Value</span>
 </div>

 <div class="card tooltip">
   <Card title="Leads/Opportunity" value={data.leadsPerOpportunity} icon="fal fa-lg fa-crosshairs" />
   <span class="tooltiptext">Avg.Leads Per Opportunity</span>
 </div>
{:else}
 <p>Loading data...</p>
{/if}
</main>
<style>

	main {
	  display: flex;
	  flex-wrap: wrap;
	  justify-content: center;
    margin-top: 50px;
	}
  .card {
    flex: 1;
    max-width: 300px;
    margin: 5px; /* Adjust this margin to control spacing */
  }
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
