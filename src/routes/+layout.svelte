<script>
	import favicon from '$lib/assets/favicon.svg';
	import '../app.scss';

	import { onMount } from 'svelte';

	let { children } = $props();

	onMount(async () => {
		const jqueryModule = await import('jquery');
		window.jQuery = jqueryModule.default;
		window.$ = window.jQuery;

		await import('bootstrap/dist/js/bootstrap.bundle.min.js');
	});
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

{@render children()}

<style>
	/* 🔥 CRITICAL GLOBAL RESET */
	:global(html, body) {
		margin: 0;
		padding: 0;
		height: 100%;
		width: 100vw;
	}
</style>