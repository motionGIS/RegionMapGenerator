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
    let gpxTraces: { id: string; color: string; data: string; name: string }[] = $state([]);

    // Reference to the StatesAndProvincesPreview component
    let mapPreviewComponent: StatesAndProvincesPreview;

    /** Handle map download */
    const handleDownload = (format: "svg" | "png") => {
        if (format === "png") {
            const map = mapPreviewComponent?.map;
            if (!map) {
                console.error("Map not loaded yet");
                return;
            }

            // Wait for the map to finish rendering
            map.once('idle', () => {
                try {
                    // Get the canvas and convert to high-res PNG
                    const canvas = map.getCanvas();
                    const dataURL = canvas.toDataURL('image/png', 1.0); // 1.0 = highest quality

                    // Create download link
                    const link = document.createElement('a');
                    link.href = dataURL;
                    link.download = `map-export-${new Date().toISOString().slice(0, 10)}.png`;
                    
                    // Trigger download
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);
                } catch (error) {
                    console.error('Error exporting map:', error);
                }
            });

            map.triggerRepaint();
        }
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
                <CardContent>
                    <GpxUploader bind:gpxTraces />
                </CardContent>
            </Card>
        <Card>
            <CardContent>
                <CardTitle>Preview</CardTitle>
                <StatesAndProvincesPreview bind:this={mapPreviewComponent} {selectedStates} {gpxTraces}/>
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