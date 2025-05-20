<script lang="ts">
	import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
	import { Button } from "$lib/components/ui/button";
	import { Input } from "$lib/components/ui/input";
	import { Label } from "$lib/components/ui/label";
	import { MapPinIcon, DownloadIcon, UploadIcon } from "lucide-svelte";
	import StateSelector from "./state-selector.svelte";
	import GpxUploader from "./gpx-uploader.svelte";
	import MapPreview from "./map-preview.svelte";

	// Selected states and GPX data
	let selectedStates: string[] = $state([]);
	let gpxTraces: { id: string; color: string; data: string }[] = $state([]);

	/** Handle map download */
	const handleDownload = (format: "svg" | "png") => {
		// Implementation for downloading map
		console.log(`Downloading map as ${format}`, { selectedStates, gpxTraces });
	};
</script>

<section id="map" class="container py-20">
	<div class="grid gap-8 lg:grid-cols-2">
		<!-- Map Controls -->
		<div class="flex flex-col gap-4">
			<Card>
				<CardHeader>
					<CardTitle>Select States</CardTitle>
				</CardHeader>
				<CardContent>
					<StateSelector bind:selectedStates />
				</CardContent>
			</Card>

			<Card>
				<CardHeader>
					<CardTitle>GPX Traces</CardTitle>
				</CardHeader>
				<CardContent>
					<GpxUploader bind:gpxTraces />
				</CardContent>
			</Card>

			<Card>
				<CardHeader>
					<CardTitle>Export Map</CardTitle>
				</CardHeader>
				<CardContent>
					<div class="flex gap-4">
						<Button onClick={() => handleDownload("svg")} disabled={selectedStates.length === 0} class="flex-1 gap-2">
							<DownloadIcon />
							Export SVG
						</Button>
						<Button onClick={() => handleDownload("png")} disabled={selectedStates.length === 0} class="flex-1 gap-2">
							<DownloadIcon />
							Export PNG
						</Button>
					</div>
				</CardContent>
			</Card>
		</div>

		<!-- Map Preview -->
		<Card>
			<CardHeader>
				<CardTitle>Map Preview</CardTitle>
			</CardHeader>
			<CardContent>
				<MapPreview {selectedStates} {gpxTraces} />
			</CardContent>
		</Card>
	</div>
</section>
