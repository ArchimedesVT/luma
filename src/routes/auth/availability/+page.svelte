<script lang="ts">
    import AvailabilityGrid from "$lib/components/applicant/AvailabilityGrid.svelte";
    import BackButton from "$lib/components/backButton.svelte";
    import { getCurrentUserEmail, supabase } from "$lib/utils/supabase";
    import { goto } from "$app/navigation";

    type Range = { date: string; start: string; end: string };
    let ranges: Range[] =  [];

    const onChange = (event: CustomEvent) => {
        ranges = event.detail.ranges;
    };

    const onSubmit = async () => {
        const newEmail = await getCurrentUserEmail();
        const { data, error } = await supabase
            .from("interviewers")
            .update({metadata: JSON.stringify(ranges) })
            .eq('email', newEmail);

        if (error) {
            console.error('Error sending application:', error);
            throw new Error('Failed to send application to Supabase');
        }
        console.log(data);

        goto(`/`);

        return data;

    }
</script>

<div class="layout">
    <div class="content">
        <h4>Select Your Availability</h4>

        <AvailabilityGrid 
            stepMinutes={15} 
            startDate="2025-09-11" 
            endDate="2025-09-15" 
            dayStart="10:00" 
            dayEnd="17:00" 
            initialRanges={ranges}
            on:change={onChange} 
        />
        
        <button on:click={onSubmit} class="btn btn-primary">Submit Availability</button>
    </div>

    <BackButton></BackButton>
</div>