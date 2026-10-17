<script lang="ts">
	import { t } from 'svelte-intl-precompile'

	import Button from '$lib/ui/Button.svelte'
	import MaxSize from '$lib/ui/MaxSize.svelte'
	import type { FileDTO } from '@cryptgeon/shared'

	interface Props {
		label?: string
		files?: FileDTO[]
		[key: string]: any
	}

	let { label = '', files = $bindable([]), ...rest }: Props = $props()

	async function fileToDTO(file: File): Promise<FileDTO> {
		return {
			name: file.name,
			mime: file.type,
			size: file.size,
			data: new Uint8Array(await file.arrayBuffer()),
		}
	}

	async function onInput(e: Event) {
		const input = e.target as HTMLInputElement
		if (input?.files?.length) {
			const toAdd = await Promise.all(Array.from(input.files).map(fileToDTO))
			files = [...files, ...toAdd]
		}
	}

	function clear(e: Event) {
		e.preventDefault()
		files = []
	}
</script>

<label>
	{#if label}
		<small>{label}</small>
	{/if}
	<input {...rest} type="file" onchange={onInput} multiple />
	<div class="box">
		{#if files.length}
			<div>
				<b>{$t('file_upload.selected_files')}</b>
				{#each files as file}
					<div class="file">
						{file.name}
					</div>
				{/each}
				<div class="spacer"></div>
				<Button onclick={clear}>{$t('file_upload.clear')}</Button>
			</div>
		{:else}
			<div>
				<svg class="up" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M12 16V4M7 9l5-5 5 5M5 20h14" /></svg>
				<b>{$t('file_upload.no_files_selected')}</b>
				<br />
				<small>
					{$t('common.max')}: <MaxSize />
				</small>
			</div>
		{/if}
	</div>
</label>

<style>
	label {
		display: block;
	}

	label > small {
		display: block;
		margin-bottom: 0.375rem;
		font-weight: 500;
	}

	input {
		display: none;
	}

	.box {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		text-align: center;
		border-style: dashed;
		border-width: 2px;
		cursor: pointer;
	}

	.box:hover {
		border-color: var(--ui-clr-primary);
	}

	.file {
		color: var(--ui-text-1);
		word-break: break-all;
	}

	.up {
		display: block;
		width: 1.75rem;
		height: 1.75rem;
		margin: 0 auto 0.5rem;
		color: var(--ui-clr-primary);
	}

	b::first-letter {
		text-transform: uppercase;
	}

	.spacer {
		margin-top: 1rem;
	}

	small::first-letter {
		text-transform: uppercase;
	}
</style>
