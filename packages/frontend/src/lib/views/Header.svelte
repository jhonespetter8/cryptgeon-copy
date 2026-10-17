<script lang="ts">
	import { status } from '$lib/stores/status'

	function reset() {
		window.location.reload()
	}
</script>

<header>
	<a onclick={reset} href="/">
		{#if $status === null}
			<!-- waiting for status to load -->
		{:else if $status.theme_image}
			<img alt="logo" src={$status.theme_image} />
		{:else}
			<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
				<rect x="4" y="11" width="16" height="10" rx="2" />
				<path d="M8 11V7a4 4 0 0 1 8 0v4" />
			</svg>
			<span>{$status.theme_page_title || 'Secure Share'}</span>
		{/if}
	</a>
</header>

<style>
	header {
		margin: 2rem 0 1.5rem;
	}

	a {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		border: none;
		font-size: 1.4rem;
		font-weight: 600;
	}

	svg {
		width: 1.6rem;
		height: 1.6rem;
		color: var(--ui-clr-primary);
	}

	img {
		max-height: 3rem;
		object-fit: contain;
	}
</style>
