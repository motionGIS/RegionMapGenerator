<script lang="ts">
	import { Button } from "$lib/components/ui/button";
	import { Input } from "$lib/components/ui/input";
	import { Label } from "$lib/components/ui/label";
	import { TrashIcon, UploadIcon } from "lucide-svelte";
	import { slide } from "svelte/transition";

	// Props
	let {
		gpxTraces = $bindable([]),
	}: {
		gpxTraces: { id: string; color: string; data: string }[];
	} = $props();

	/** Handle file upload */
	const handleFileUpload = async (event: Event) => {
		const input = event.target as HTMLInputElement;
		const file = input.files?.[0];

		if (file) {
			const reader = new FileReader();
			reader.onload = (e) => {
				const data = e.target?.result as string;
				gpxTraces = [
					...gpxTraces,
					{
						id: crypto.randomUUID(),
						color: "#FF0000", // Default red color
						data,
					},
				];
			};
			reader.readAsText(file);
		}
	};

	/** Update trace color */
	const updateTraceColor = (id: string, color: string) => {
		gpxTraces = gpxTraces.map((trace) => (trace.id === id ? { ...trace, color } : trace));
	};

	/** Remove trace */
	const removeTrace = (id: string) => {
		gpxTraces = gpxTraces.filter((trace) => trace.id !== id);
	};
</script>

<div class="flex flex-col gap-4">
	<div class="grid w-full max-w-sm items-center gap-1.5">
		<Label for="gpx">Upload GPX File</Label>
		<Input id="gpx" type="file" accept=".gpx" onChange={handleFileUpload} />
	</div>

	<div class="flex flex-col gap-2">
		{#each gpxTraces as trace}
			<div transition:slide class="flex items-center gap-2">
				<Input type="color" value={trace.color} onInput={(e) => updateTraceColor(trace.id, (e.target as HTMLInputElement).value)} class="h-8 w-12" />
				<span class="flex-1 truncate text-sm">
					GPX Trace {trace.id.slice(0, 8)}
				</span>
				<Button size="icon" variant="destructive" onClick={() => removeTrace(trace.id)}>
					<TrashIcon class="h-4 w-4" />
				</Button>
			</div>
		{/each}
	</div>
</div>
