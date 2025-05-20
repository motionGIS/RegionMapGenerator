<script lang="ts">
	import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
	import { Button } from "$lib/components/ui/button";
	import { Input } from "$lib/components/ui/input";
	import { Label } from "$lib/components/ui/label";
	import { MapPinIcon, DownloadIcon, UploadIcon } from "lucide-svelte";
	import StateSelector from "./state-selector.svelte";
	import GpxUploader from "./gpx-uploader.svelte";
	import MapPreview from "./map-preview.svelte";
    import StatesAndProvincesPreview from "./states-and-provinces-preview.svelte";

	// Selected states and GPX data
	let selectedStates: string[] = $state([]);
	let gpxTraces: { id: string; color: string; data: string }[] = $state([]);

	/** Handle map download */
	const handleDownload = (format: "svg" | "png") => {
		// Implementation for downloading map
		console.log(`Downloading map as ${format}`, { selectedStates, gpxTraces });
	};
</script>

<section id="map" class="container py-10">
	<div class="grid gap-4 lg:grid-cols-2">
		<!-- Map Controls -->
		<div class="flex flex-col gap-3">
			<Card>
				<CardHeader>
					<CardTitle>Select States/Provinces/Territories</CardTitle>
				</CardHeader>
				<CardContent>
					<StateSelector bind:selectedStates />
				</CardContent>
			</Card>
		</div>

		<div class="flex flex-col gap-3">
			<Card>
				<CardHeader>
					<CardTitle>GPX Traces</CardTitle>
				</CardHeader>
				<CardContent>
					<GpxUploader bind:gpxTraces />
				</CardContent>
			</Card>
		<Card>
			<CardContent>
				<CardTitle>Preview</CardTitle>
				<StatesAndProvincesPreview {selectedStates} />
					<div class="flex gap-4">
						<Button onclick={() => handleDownload("png")} disabled={selectedStates.length === 0} class="flex-1 gap-2 {selectedStates.length === 0 ? '' :'cursor-pointer'}">
							<DownloadIcon />
							Export PNG
						</Button>
					</div>
			</CardContent>
		</Card>
		</div>
	</div>
</section>
