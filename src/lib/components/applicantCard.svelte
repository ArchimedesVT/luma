<script lang="ts">
    import { onMount } from 'svelte';
    import { getAllApplicants } from '$lib/utils/supabase';
    import { goto } from '$app/navigation';

    let applicants: {
        id: number;
        name: string;
        created_at: string;
        comments: {
            id: number;
            email: string;
            comment: string;
            decision: string;
        }[];
    }[] = [];

    type Applicant = {
        id: number;
        name: string;
        created_at: string;
        comments: {
            id: number;
            email: string;
            comment: string;
            decision: string;
        }[];
    } | null;

    type Comment = {
        id: number;
        email: string;
        comment: string;
        decision: string;
    };

    let searchQuery: string = ''; // New variable for search input

    onMount(async () => {
        try {
            applicants = await getAllApplicants();
        } catch (error) {
            console.error('Failed to load applicants:', error);
        }
    });

    const navigateToReview = (id: number) => {
        goto(`/private/recruiter/review/candidate?id=${id}`);
    };

    // Computed property to filter applicants based on search query
    $: filteredApplicants = applicants.filter(applicant =>
        applicant.name.toLowerCase().includes(searchQuery.toLowerCase())
    );

    // Toggle for the button
    let commentsBool: boolean = false;
    // Threshold for amount of required comments
    let commentsThreshold: number = 2;

    const getAcceptedCount = (newApplicant: Applicant) => {
        let count: number = 0;
        let newComments: {
            id: number;
            email: string;
            comment: string;
            decision: string;
        }[] = newApplicant ? newApplicant.comments.comments : [];

        if (newApplicant && newApplicant.comments) {
            for (const comment of newComments) {
                if (comment.decision == "Approved") {
                    count++;
                }
            }
        }
        return count;
    };

    // Filter applicants when selected 
    async function filterApplicants(): Promise<boolean> {
        commentsBool = !commentsBool;

        if (commentsBool) {
            filteredApplicants = applicants.filter(applicant =>
                applicant.comments.comments.length <= commentsThreshold
            );
        } else {
            try {
                applicants = await getAllApplicants();
            } catch (error) {
                console.error('Failed to load applicants:', error);
            }

            filteredApplicants = applicants.filter(applicant =>
                applicant.name.toLowerCase().includes(searchQuery.toLowerCase())
            );
        }

        return commentsBool;
    }
</script>

<div class="mb-4">
    <input
        type="text"
        placeholder="Search by name..."
        bind:value={searchQuery}
        class="w-full border border-gray-300 rounded p-2"
    />

    <button class="btn btn-primary" on:click={filterApplicants}>
        Show Which Require Review
    </button>
</div>

<div class="grid grid-cols-4 gap-4">
    {#each filteredApplicants as applicant}
        <div
            class="border border-gray-300 rounded-lg p-4 shadow cursor-pointer"
            on:click={() => navigateToReview(applicant.id)}
        >
            <h2 class="text-sm mb-2 text-black">{applicant.name}</h2>
            <p class="text-sm text-gray-600">
                Submitted at: {new Date(applicant.created_at).toLocaleString()}
            </p>
            <p>Comments Count: {applicant.comments.comments.length}</p>
            <p>Accepted Count: {getAcceptedCount(applicant)}</p>
        </div>
    {/each}
</div>
