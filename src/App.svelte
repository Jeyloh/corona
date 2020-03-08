<script>
	import Map from "./Map.svelte";
	import { count } from "./stores.js";
	import { onDestroy } from "svelte";
	export let name;

	function increment () {
		$count += 1;
	}
	function decrement () {
		$count -= 1;
	}


</script>

<main>
	<Map />

</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>