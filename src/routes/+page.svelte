<script lang="ts">
	import Nav from "$lib/components/nav/nav.svelte";
	import MapSection from "$lib/components/map/map-section.svelte";
	import Footer from "$lib/components/footer.svelte";
</script>

<div class="min-h-dvh bg-background text-foreground">
	<Nav />
	<main class="md:mx-16">
		<MapSection />
	</main>
	<Footer />
</div>
