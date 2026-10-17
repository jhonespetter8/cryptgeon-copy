<script lang="ts">
	import { status } from '$lib/stores/status'
</script>

<footer>
	<small>Encrypted in your browser. The server never sees the key.</small>
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
		flex-wrap: wrap;
		justify-content: space-between;
		align-items: center;
		gap: 0.5rem 1.5rem;
		width: 100%;
		max-width: 40rem;
		margin: 0 auto;
		padding: 2rem 1rem 2.5rem;
	}

	nav {
		display: flex;
		gap: 1.25rem;
		font-size: 0.8125rem;
		color: var(--ui-text-1);
	}

	a {
		text-decoration: none;
	}

	a:hover {
		color: var(--ui-text-0);
	}
</style>
