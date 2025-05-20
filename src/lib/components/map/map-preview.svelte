<script lang="ts">
	// Props
	let {
		selectedStates = [],
		gpxTraces = [],
	}: {
		selectedStates: string[];
		gpxTraces: { id: string; color: string; data: string }[];
	} = $props();

	// SVG map data would go here
	// This would contain path data for each state
	const statesPaths: Record<string, string> = {
		AL: "M...", // Path data for Alabama
		AK: "M...", // Path data for Alaska
		// ... Add all states
	};
</script>

<div class="relative aspect-[4/3] w-full overflow-hidden rounded-lg border bg-muted">
	<svg viewBox="0 0 959 593" class="h-full w-full">
		<!-- Render states -->
		{#each Object.entries(statesPaths) as [stateCode, pathData]}
			<path d={pathData} class="transition-colors duration-200" class:opacity-20={selectedStates.length > 0 && !selectedStates.includes(stateCode)} fill={selectedStates.includes(stateCode) ? "currentColor" : "#e2e8f0"} stroke="white" stroke-width="1" />
		{/each}

		<!-- Render GPX traces -->
		{#each gpxTraces as trace}
			<!-- Parse and render GPX data as SVG path -->
			<path d="M..." fill="none" stroke={trace.color} stroke-width="2" />
		{/each}
	</svg>
</div>
