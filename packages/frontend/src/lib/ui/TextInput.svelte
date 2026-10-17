<script lang="ts">
	import Icon from '$lib/ui/Icon.svelte'
	import { copy as copyFN } from '$lib/utils'
	import { randomBytes, bytesToHex } from '@cryptgeon/shared'
	import type { HTMLInputAttributes } from 'svelte/elements'

	interface Props {
		label?: string
		value: any
		validate?: (value: any) => boolean | string
		copy?: boolean
		random?: boolean
	}

	let {
		label = '',
		value = $bindable(),
		validate = () => true,
		copy = false,
		random = false,
		...rest
	}: HTMLInputAttributes & Props = $props()

	// svelte-ignore state_referenced_locally
	const initialType = $state(rest.type)
	const isPassword = initialType === 'password'
	let hidden = $state(true)

	let valid = $derived(validate(value))
	let type = $derived(isPassword ? (hidden ? 'password' : 'text') : rest.type)

	function toggle() {
		console.debug('toggle')
		hidden = !hidden
	}

	async function randomFN() {
		value = bytesToHex(randomBytes(32))
	}
</script>

<label>
	<small class:disabled={rest.disabled}>
		{label}
		{#if valid !== true}
			<span class="error-text">{valid}</span>
		{/if}
	</small>
	<input bind:value {...rest} {type} autocomplete="off" class:valid={valid === true} />
	<div class="icons">
		{#if isPassword}
			<Icon
				disabled={rest.disabled}
				class="icon"
				icon={hidden ? 'eye' : 'eye-off'}
				onclick={toggle}
			/>
		{/if}
		{#if random}
			<Icon disabled={rest.disabled} class="icon" icon="dice" onclick={randomFN} />
		{/if}
		{#if copy}
			<Icon
				disabled={rest.disabled}
				class="icon"
				icon="copy"
				onclick={() => copyFN(value.toString())}
			/>
		{/if}
	</div>
</label>

<style>
	label {
		position: relative;
		display: block;
		width: 100%;
	}

	label > small {
		display: block;
		margin-bottom: 0.375rem;
		font-weight: 500;
	}

	input {
		width: 100%;
		margin: 0;
		border: 1px solid var(--ui-bg-1);
		background: var(--ui-bg-0);
		outline: none;
		padding: 0.5rem 0.75rem;
		height: 2.5rem;
	}

	input:hover {
		border-color: var(--ui-bg-2);
	}

	input:focus {
		border-color: var(--ui-clr-primary);
		box-shadow: var(--ui-ring);
	}

	input:not(.valid) {
		border-color: var(--ui-clr-error);
	}

	.icons {
		position: absolute;
		right: 0.35rem;
		bottom: 0.35rem;
		display: flex;
		gap: 0.25rem;
		color: var(--ui-text-1);
	}

	.icons > :global(.icon) {
		width: 1.25rem;
		height: 1.25rem;
		padding: 0.3rem;
		border-radius: 6px;
		cursor: pointer;
	}

	.icons > :global(.icon:hover) {
		background-color: var(--ui-bg-1);
		color: var(--ui-clr-primary);
	}

	small::first-letter {
		text-transform: uppercase;
	}
</style>
