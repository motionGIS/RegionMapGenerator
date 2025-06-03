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

    const handleDownload = (resolution: 2 | 4 | 8) => {
        const map = mapPreviewComponent?.map;
        if (!map) {
            console.error("Map not loaded yet");
            return;
        }

        // Wait for the map to finish rendering
        map.once('idle', () => {
            try {
                const canvas = map.getCanvas();
                const originalPixelRatio = window.devicePixelRatio;
                
                // Set pixel ratio for desired resolution
                Object.defineProperty(window, 'devicePixelRatio', {
                    get() { return resolution; }
                });

                // Get original canvas size
                const originalWidth = canvas.width;
                const originalHeight = canvas.height;

                // Resize canvas to new resolution
                map.getCanvasContainer().style.width = canvas.style.width;
                map.getCanvasContainer().style.height = canvas.style.height;
                
                // Trigger a resize to apply the new pixel ratio
                map.resize();

                // Wait for the high-res render to complete
                map.once('render', () => {
                    try {
                        // Export the high-resolution canvas
                        const dataURL = canvas.toDataURL('image/png', 1.0);

                        // Restore original pixel ratio
                        Object.defineProperty(window, 'devicePixelRatio', {
                            get() { return originalPixelRatio; }
                        });

                        // Resize back to original
                        map.resize();

                        // Create download link
                        const link = document.createElement('a');
                        link.href = dataURL;
                        link.download = `map-export-${resolution}x-${new Date().toISOString().slice(0, 10)}.png`;
                        
                        // Trigger download
                        document.body.appendChild(link);
                        link.click();
                        document.body.removeChild(link);
                    } catch (error) {
                        console.error(`Error exporting ${resolution}x map:`, error);
                        
                        // Restore original pixel ratio on error
                        Object.defineProperty(window, 'devicePixelRatio', {
                            get() { return originalPixelRatio; }
                        });
                        map.resize();
                    }
                });

                // Trigger re-render at high resolution
                map.triggerRepaint();
            } catch (error) {
                console.error('Error setting up high-res export:', error);
            }
        });

        map.triggerRepaint();
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
                    <div class="flex flex-col md:flex-row gap-2">
                        <Button onclick={() => handleDownload(2)} disabled={selectedStates.length === 0} class="flex-1 gap-2">
                            <DownloadIcon />
							Export PNG (1x res) 
                        </Button>
                        <Button onclick={() => handleDownload(4)} disabled={selectedStates.length === 0} class="flex-1 gap-2">
                            <DownloadIcon />
                            Export PNG (2x res) 
                        </Button>
                        <Button onclick={() => handleDownload(8)} disabled={selectedStates.length === 0} class="flex-1 gap-2">
                            <DownloadIcon />
                            Export PNG (4x res)
                        </Button>
                    </div>
            </CardContent>
        </Card>
        </div>
    </div>
</section>