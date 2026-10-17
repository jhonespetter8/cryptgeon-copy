<script lang="ts">
	import type { HTMLButtonAttributes } from 'svelte/elements'

	interface Props {
		children?: import('svelte').Snippet
	}

	let { children, ...rest }: HTMLButtonAttributes & Props = $props()
</script>

<button {...rest} class:primary={rest.type === 'submit'}>{@render children?.()}</button>

<style>
	button {
		display: inline-flex;
		text-transform: capitalize;
		align-items: center;
		justify-content: center;
		padding: 0 1.25rem;
		font-weight: 600;
		height: 2.5rem;
		border: 1px solid var(--ui-bg-2);
		background: var(--ui-surface);
		outline: none;
		cursor: pointer;
	}

	button:hover {
		border-color: var(--ui-text-1);
	}

	button:focus-visible {
		box-shadow: var(--ui-ring);
	}

	.primary {
		background: var(--ui-clr-primary);
		border-color: var(--ui-clr-primary);
		color: var(--ui-clr-on-primary);
		padding: 0 1.75rem;
	}

	.primary:hover {
		background: var(--ui-clr-primary-alt);
		border-color: var(--ui-clr-primary-alt);
	}
</style>
