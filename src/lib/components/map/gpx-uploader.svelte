<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Label } from "$lib/components/ui/label";
    import { MoveLeft, TrashIcon } from "lucide-svelte";
    import { slide, fade, fly } from "svelte/transition";

    let {
        gpxTraces = $bindable([]),
    }: {
        gpxTraces: { id: string; color: string; data: string; name: string }[];
    } = $props();

    const MAX_TRACES = 30;

    /** Handle file upload */
    const handleFileUpload = async (event: Event) => {
        const input = event.target as HTMLInputElement;
        const files = input.files;

        if (!files) return;

        // Only add up to the max allowed
        const availableSlots = MAX_TRACES - gpxTraces.length;
        const filesToAdd = Array.from(files).slice(0, availableSlots);

        for (const file of filesToAdd) {
            const reader = new FileReader();
            reader.onload = (e) => {
                const data = e.target?.result as string;
                gpxTraces = [
                    ...gpxTraces,
                    {
                        id: crypto.randomUUID(),
                        color: "#FF0000",
                        data,
                        name: file.name,
                    },
                ];
            };
            reader.readAsText(file);
        }

        // Reset the input so the same file(s) can be uploaded again if needed
        input.value = "";
    };

    const updateTraceColor = (id: string, color: string) => {
        gpxTraces = gpxTraces.map((trace) => (trace.id === id ? { ...trace, color } : trace));
    };

    const removeTrace = (id: string) => {
        gpxTraces = gpxTraces.filter((trace) => trace.id !== id);
    };
</script>

<div class="flex flex-col gap-4">
    <div class="grid w-full max-w-sm items-center gap-1.5">
        <Label for="gpx">Upload GPX Files</Label>
        <Input
            id="gpx"
            type="file"
            multiple
            accept=".gpx"
            onchange={handleFileUpload}
            disabled={gpxTraces.length >= MAX_TRACES}
        />
        {#if gpxTraces.length >= MAX_TRACES}
            <span class="text-xs text-red-500">Maximum of {MAX_TRACES} traces reached.</span>
        {/if}
    </div>

    <div class="flex flex-col gap-2">
        {#each gpxTraces as trace (trace.id)}
            <div transition:fly={{ y: -50, duration: 150 }} class="flex items-center gap-2">
                <Input
                    type="color"
                    value={trace.color}
                    oninput={(e) => updateTraceColor(trace.id, (e.target as HTMLInputElement).value)}
                    class="h-8 w-12 cursor-pointer"
                />
                <span class="flex-1 truncate text-sm">
                    {trace.name}
                </span>
                <Button class="cursor-pointer" size="icon" variant="destructive" onclick={() => removeTrace(trace.id)}>
                    <TrashIcon class="h-4 w-4 " />
                </Button>
            </div>
        {/each}
    </div>
</div>