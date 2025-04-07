<script>
import * as d3 from "d3";
import { onMount } from "svelte";

let data = [];

onMount(async () => {
	data = await d3.csv("/loc.csv");
});

</script>
<svelte:head>
<title>Meta</title>
</svelte:head>
<h1>Meta</h1>
<p>This page includes stats about the code of this website</p>
<p>Total lines of code: {data.length}</p>