<script lang="ts">
	import ThemeToggle from '$lib/ui/ThemeToggle.svelte'
	import { status } from '$lib/stores/status'
</script>

<footer>
	<ThemeToggle />
	<nav>
		{#if $status?.theme_home_link !== false}
			<a href="/">Home</a>
		{/if}
		<a href="/about">About</a>
		{#if $status?.imprint_url}
			<a href={$status.imprint_url} target="_blank" rel="noopener noreferrer">Imprint</a>
		{:else if $status?.imprint_html}
			<a href="/imprint">Imprint</a>
		{/if}
	</nav>
</footer>

<style>
	footer {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0.75rem 1rem;
		position: fixed;
		bottom: 0;
		right: 0;
		width: 100%;
		font-size: 0.875rem;
		color: var(--ui-text-1);
		background-color: var(--ui-bg-0-85);
		backdrop-filter: blur(8px);
		border-top: 1px solid var(--ui-bg-1);
	}

	a {
		margin: 0 0.5rem;
		border: none;
	}

	a:hover {
		color: var(--ui-text-0);
	}

	nav {
		display: flex;
		justify-content: flex-end;
		align-items: center;
	}
</style>
