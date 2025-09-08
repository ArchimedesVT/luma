<script lang="ts">
	import Sidebar from "$lib/components/recruiter/Sidebar.svelte";
  import Navbar from "$lib/components/recruiter/Navbar.svelte";
  import type { PageData } from './$types'; // Import the correct type for props
  import { goto } from "$app/navigation";

  export let data: PageData; // Declare the data prop
  const user = data.user; // Access user directly from data
</script>

<div class="layout">
  <div class="content-left">
    <h4 style="text-align: left;">Hello,  {user?.email}</h4>

    <p>Utilize the navbar to the left to navigate around luma.</p>

    <button on:click={() => goto('/private/availability')} class="availability-button styled-button">
      Update your availability
    </button>

  </div>

  <Navbar/>

  <Sidebar currentStep={0}/>
</div>

<style>
  .styled-button {
    background-color: #007bff; /* Bootstrap primary color */
    color: white;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .styled-button:hover {
    background-color: #0056b3; /* Darker shade for hover effect */
  }
</style>