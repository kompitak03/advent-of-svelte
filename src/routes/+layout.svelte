<script>
	import '../app.css';
	import Header from '$lib/header/+page.svelte';
	import { page } from '$app/stores';
</script>

{#if $page.url.pathname !== '/'}
	<Header />
{/if}
<slot />
