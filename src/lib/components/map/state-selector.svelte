<script lang="ts">
    import StatesAndProvinces from "./states-and-provinces.svelte";
	export let selectedStates: string[] = [];
</script>

<div class="flex flex-col gap-4 w-full h-full">
	<StatesAndProvinces bind:selectedStates/>
</div>
