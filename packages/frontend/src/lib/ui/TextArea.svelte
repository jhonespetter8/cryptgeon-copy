<script lang="ts">
	interface Props {
		label?: string
		value: string
		[key: string]: any
	}

	let { label = '', value = $bindable(), ...rest }: Props = $props()
</script>

<label>
	{#if label}
		<small class="label">{label}</small>
	{/if}
	<textarea class="box" {...rest} bind:value></textarea>
</label>

<style>
	label {
		display: block;
	}

	.label {
		display: block;
		margin-bottom: 0.375rem;
		font-weight: 500;
	}

	small::first-letter {
		text-transform: uppercase;
	}
</style>
