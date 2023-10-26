<!-- Card.svelte -->
<script>
    export let title;
    export let value;
    export let icon;
  </script>
  
  <div class="card">
    {#if icon}
      <div class="icon">
        <i class="fas {icon}"></i> <!-- Use Font Awesome icon class -->
        <div class="value">{value}</div>
      </div>
    {/if}
    <div class="title text-muted">{title}</div>
  </div>
  
  <style>
    .card {
      border: 1px solid #ccc;
      padding: 10px;
      text-align: center;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
  
    .icon {
      display: flex;
      flex-direction: row; /* Display icon and value side by side */
      align-items: center; /* Center icon and value vertically */
    }
  
    .icon i {
      font-size: 15px;
      margin-right: 0px; /* Add spacing between icon and value */
    }
  
    .value {
      font-size: 18px; /* You can adjust the font size as needed */
    }
  
    .title {
      font-weight: bolder;
      
    }
  
    .text-muted {
      font-weight: 600;
      font-style: normal;
      line-height: 20px;
      font-size: 12px;
      color: #6c798f !important;
    }
  </style>
  
