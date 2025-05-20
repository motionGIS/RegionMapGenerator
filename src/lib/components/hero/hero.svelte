<script lang="ts">
	import { Button } from "$lib/components/ui/button";
	import { MapIcon } from "lucide-svelte";
</script>

<section id="hero" class="container py-20 text-center">
	<h1 class="text-4xl font-bold sm:text-5xl md:text-6xl">Create Custom US State Maps</h1>
	<p class="mx-auto mt-4 max-w-2xl text-muted-foreground">Select states, upload GPX traces, and generate custom maps for your needs. Download as SVG or PNG with your selected states and traces.</p>
	<div class="mt-8">
		<Button href="#map" size="lg" class="gap-2">
			<MapIcon />
			Start Creating
		</Button>
	</div>
</section>
