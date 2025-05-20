<script lang="ts">
	import Nav from "$lib/components/nav/nav.svelte";
	import Hero from "$lib/components/hero/hero.svelte";
	import MapSection from "$lib/components/map/map-section.svelte";
	import Footer from "$lib/components/footer.svelte";
</script>

<div class="min-h-dvh bg-background text-foreground">
	<Nav />
	<main>
		<Hero />
		<MapSection />
	</main>
	<Footer />
</div>
