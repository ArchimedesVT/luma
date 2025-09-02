<script lang="ts">
  import Sidebar from "$lib/components/applicant/Sidebar.svelte";
  import Navbar from "$lib/components/applicant/Navbar.svelte";
  import Footer from "$lib/components/applicant/Footer.svelte";
  import AvailabilityGrid from "$lib/components/applicant/AvailabilityGrid.svelte";

  type Range = { date: string; start: string; end: string };
  let ranges: Range[] = JSON.parse(localStorage.getItem('availability') || '[]');
  
  const onChange = (event: CustomEvent) => {
    ranges = event.detail.ranges;
    localStorage.setItem('availability', JSON.stringify(ranges)); // Store as JSON string
  }; 

</script>

<div class="layout">
  <div class="content">
    <h4>Select Your Availability</h4>

    <AvailabilityGrid 
      stepMinutes={30} 
      startDate="2025-09-11" 
      endDate="2025-09-15" 
      dayStart="09:00" 
      dayEnd="21:00" 
      initialRanges={ranges}
      on:change={onChange} 
      disabledRanges={[{ date: '2025-09-11', start: '09:00', end: '17:00' }, {date: '2025-09-12', start:'09:00', end: '17:00'}, {date:'2025-09-15', start:'09:00', end:'17:00'}]}
  />

    <Footer backNav="/applicant/3_teams" nextNav="/applicant/5_free_response"/>
  </div>

  <Navbar/>

  <Sidebar currentStep={3}/>
</div>