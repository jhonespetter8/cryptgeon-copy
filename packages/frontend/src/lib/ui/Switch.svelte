<script lang="ts">
	interface Props {
		label?: string
		value: boolean
		color?: boolean
		[key: string]: any
	}

	let { label = '', value = $bindable(), color = true, ...rest }: Props = $props()
</script>

<label {...rest}>
	<input type="checkbox" bind:checked={value} />
	<span class:color class="slider"></span>
	<small>{label}</small>
</label>

<style>
	label {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		height: 2.5rem;
		cursor: pointer;
		user-select: none;
	}

	label input {
		display: none;
	}

	small {
		width: max-content;
	}

	.slider {
		position: relative;
		flex-shrink: 0;
		width: 2.25rem;
		height: 1.25rem;
		border-radius: 999px;
		background-color: var(--ui-bg-2);
		transition: var(--ui-anim);
	}

	.slider:before {
		position: absolute;
		content: '';
		width: 1rem;
		height: 1rem;
		left: 0.125rem;
		top: 0.125rem;
		border-radius: 50%;
		background-color: #fff;
		box-shadow: 0 1px 2px rgb(0 0 0 / 0.25);
		transition: var(--ui-anim);
	}

	input:checked + .slider.color {
		background-color: var(--ui-clr-primary);
	}

	input:checked + .slider:before {
		transform: translateX(1rem);
	}
</style>
