<script lang="ts">
  import Sidebar from "$lib/components/recruiter/Sidebar.svelte";
  import Navbar from "$lib/components/recruiter/Navbar.svelte";
  import ScheduleGrid from "$lib/components/recruiter/ScheduleGrid.svelte";
  import { supabase } from "$lib/utils/supabase";
  import { onMount } from 'svelte';

  let newData;
  type Range = { date: string; start: string; end: string };

  let interviewRanges: Range[] = [];
  let interviewStrings: string[] = [];

  onMount(async () => {
    const { data, error } = await supabase
      .from("interviews")
      .select('*')
      .eq('interviewer', (await supabase.auth.getUser()).data.user?.email);

    if (error) {
      console.error('Error sending application:', error);
      throw new Error('Failed to send application to Supabase');
    }
    console.log(data);
    newData = data;

    let i: number = 0;

    // Process newData to fill interviewRanges and interviewStrings
    interviewRanges = newData.map(interview => {
      const interviewType = interview.type;
      const interviewRoom = interview.location;

      const fullString = `${interviewType} : ${interviewRoom}`;
      interviewStrings[i] = fullString;
      i++;

      const startDate = new Date(interview.startTime);
      const endDate = new Date(interview.endTime);

      return {
        date: startDate.toISOString().split('T')[0], // Extract date in YYYY-MM-DD format
        start: startDate.toISOString().split('T')[1].substring(0, 5), // Extract time in HH:mm format
        end: endDate.toISOString().split('T')[1].substring(0, 5) // Extract time in HH:mm format
      };
    });
  });
</script>

<div class="layout">
  <div class="content-left">
    <h4 style="text-align: left;">Full Schedule</h4>
    <ScheduleGrid
      stepMinutes={30}
      startDate="2025-09-11"
      endDate="2025-09-15"
      dayStart="09:00"
      dayEnd="21:00"
      appointmentRanges={interviewRanges}
      appointmentText={interviewStrings}
    />
  </div>

  <Navbar />
  <Sidebar currentStep={2} />
</div>