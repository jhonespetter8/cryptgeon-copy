<script lang="ts">
	import { status } from '$lib/stores/status'
	import ThemeToggle from '$lib/ui/ThemeToggle.svelte'

	function reset() {
		window.location.reload()
	}
</script>

<header>
	<a class="brand" onclick={reset} href="/">
		{#if $status === null}
			<!-- waiting for status to load -->
		{:else if $status.theme_image}
			<img alt="logo" src={$status.theme_image} />
		{:else}
			<span class="mark">
				<svg
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2.2"
					stroke-linecap="round"
					stroke-linejoin="round"
					aria-hidden="true"
				>
					<rect x="5" y="11" width="14" height="9" rx="2" />
					<path d="M8.5 11V8a3.5 3.5 0 0 1 7 0v3" />
				</svg>
			</span>
			<span>{$status.theme_page_title || 'Secure Share'}</span>
		{/if}
	</a>
	<ThemeToggle />
</header>

<style>
	header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
		padding: 1.5rem 0 2.75rem;
	}

	.brand {
		display: inline-flex;
		align-items: center;
		gap: 0.625rem;
		font-family: var(--font-display);
		font-weight: 650;
		font-size: 1.0625rem;
		letter-spacing: -0.01em;
		text-decoration: none;
	}

	.mark {
		display: grid;
		place-items: center;
		width: 2rem;
		height: 2rem;
		border-radius: 9px;
		background: var(--ui-clr-primary);
		color: var(--ui-clr-on-primary);
	}

	svg {
		width: 1.125rem;
		height: 1.125rem;
	}

	img {
		max-height: 2.5rem;
		object-fit: contain;
	}
</style>
