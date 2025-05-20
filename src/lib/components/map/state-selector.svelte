<script lang="ts">
	import { Checkbox } from "$lib/components/ui/checkbox";
	import { Input } from "$lib/components/ui/input";
	import { slide } from "svelte/transition";

	// Props
	let { selectedStates = $bindable([]) }: { selectedStates: string[] } = $props();

	// US States data
	const usStates = [
		{ code: "AL", name: "Alabama" },
		{ code: "AK", name: "Alaska" },
		// ... Add all states
	];

	// Search filter
	let searchTerm = $state("");

	// Filtered states based on search
	const filteredStates = $derived(usStates.filter((state) => state.name.toLowerCase().includes(searchTerm.toLowerCase()) || state.code.toLowerCase().includes(searchTerm.toLowerCase())));

	/** Toggle state selection */
	const toggleState = (stateCode: string) => {
		if (selectedStates.includes(stateCode)) {
			selectedStates = selectedStates.filter((code) => code !== stateCode);
		} else {
			selectedStates = [...selectedStates, stateCode];
		}
	};
</script>

<div class="flex flex-col gap-4">
	<Input type="search" placeholder="Search states..." bind:value={searchTerm} />

	<div class="grid max-h-[400px] gap-2 overflow-y-auto sm:grid-cols-2">
		{#each filteredStates as state}
			<div transition:slide class="flex items-center space-x-2">
				<Checkbox id={state.code} checked={selectedStates.includes(state.code)} onCheckedChange={() => toggleState(state.code)} />
				<label for={state.code} class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70">
					{state.name}
				</label>
			</div>
		{/each}
	</div>
</div>
