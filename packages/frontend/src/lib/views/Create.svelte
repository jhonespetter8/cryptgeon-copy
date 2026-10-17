<script lang="ts">
	import {
		create as apiCreate,
		bytesToHex,
		packContent,
		type FileDTO,
		type NoteInput,
		type ServerNote,
	} from '@cryptgeon/shared'
	import { t } from 'svelte-intl-precompile'
	import { slide } from 'svelte/transition'
	import { transfer } from 'comlink'

	import { status } from '$lib/stores/status'
	import { notify } from '$lib/toast'
	import AdvancedParameters from '$lib/ui/AdvancedParameters.svelte'
	import Button from '$lib/ui/Button.svelte'
	import FileUpload from '$lib/ui/FileUpload.svelte'
	import Loader from '$lib/ui/Loader.svelte'
	import MaxSize from '$lib/ui/MaxSize.svelte'
	import Result, { type NoteResult } from '$lib/ui/NoteResult.svelte'
	import PastedFilesPreview from '$lib/ui/PastedFilesPreview.svelte'
	import Switch from '$lib/ui/Switch.svelte'
	import TextArea from '$lib/ui/TextArea.svelte'
	import { createWorker } from '$lib/worker'
	import { onMount } from 'svelte'

	let note: { views: number; expiration: number } = $state({ views: 1, expiration: 60 })
	let files: FileDTO[] = $state([])
	let result: NoteResult | null = $state(null)
	let advanced = $state(false)
	let isFile = $state(false)
	let timeExpiration = $state(false)
	let customPassword: string | null = $state(null)
	let description = $state('')
	let loading: string | null = $state(null)
	let isPasting = $state(false)
	let textContent = $state('')

	$effect(() => {
		if (!advanced) {
			note.views = 1
			timeExpiration = false
		}
	})

	$effect(() => {
		description = $t('home.explanation', {
			values: {
				type: $t(timeExpiration ? 'common.minutes' : 'common.views', {
					values: { n: (timeExpiration ? note.expiration : note.views) ?? '?' },
				}),
			},
		})
	})

	$effect(() => {
		if (!isFile) textContent = ''
	})

	const worker = createWorker()

	async function handlePaste(e: ClipboardEvent) {
		const data = e.clipboardData
		if (!data) return

		const raw: File[] = []

		if (data.files.length) {
			raw.push(...Array.from(data.files))
		}

		for (let i = 0; i < data.items.length; i++) {
			const item = data.items[i]
			if (item.kind === 'file') {
				const file = item.getAsFile()
				if (file) raw.push(file)
			}
		}

		if (raw.length === 0) return
		e.preventDefault()

		const seen = new Set<string>()
		const pasted: File[] = []
		for (const f of raw) {
			const key = `${f.name}|${f.size}`
			if (!seen.has(key)) {
				seen.add(key)
				pasted.push(f)
			}
		}

		isPasting = true

		const dtos: FileDTO[] = await Promise.all(
			pasted.map(async (file) => {
				const ext = file.name.includes('.') ? '' : `.${file.type.split('/')[1] || 'bin'}`
				const name =
					file.name || `pasted-file-${Date.now()}-${Math.round(Math.random() * 1000)}${ext}`
				const renamed = new File([file], name, { type: file.type })
				const data = new Uint8Array(await renamed.arrayBuffer())
				return {
					name: renamed.name,
					mime: renamed.type,
					size: renamed.size,
					data,
				}
			})
		)

		if (dtos.length > 0) {
			if (!isFile) isFile = true
			files = [...files, ...dtos]
		}

		isPasting = false
	}

	class EmptyContentError extends Error {}

	async function submit(e: SubmitEvent) {
		e.preventDefault()
		try {
			loading = $t('common.encrypting')

			if (isFile) {
				if (files.length === 0) throw new EmptyContentError()
			} else if (textContent === '') {
				throw new EmptyContentError()
			}

			const noteInput: NoteInput = isFile
				? transfer(
						{
							type: 'files',
							files: $state.snapshot(files),
						},
						files.map((f) => f.data.buffer)
					)
				: { type: 'text', text: textContent }
			const payload = await worker.pack(noteInput, customPassword || undefined)
			const serverNote: ServerNote = {
				meta: {
					...(timeExpiration
						? { expiration: parseInt(note.expiration as any) }
						: { views: parseInt(note.views as any) }),
					extra: payload.extra,
				},
				data: payload.data,
			}

			loading = $t('common.uploading')
			const response = await apiCreate(serverNote)
			result = {
				id: response.id,
				password: customPassword ? undefined : bytesToHex(payload.key),
			}
			notify.success($t('home.messages.note_created'))
		} catch (e) {
			console.error(e)
			notify.error($t('home.errors.note_error'))
		} finally {
			loading = null
		}
	}
</script>


{#if result}
	<div class="sheet">
		<Result {result} />
	</div>
{:else}
	<section class="hero">
		<h1>Share a secret, once.</h1>
		<p class="intro">
			{@html $status?.theme_text || $t('home.intro')}
		</p>
	</section>

	<form class="composer" onsubmit={submit} onpaste={handlePaste}>
		<fieldset disabled={loading !== null}>
			{#if $status?.allow_files}
				<div class="tabs" role="tablist">
					<button
						type="button"
						role="tab"
						aria-selected={!isFile}
						class:active={!isFile}
						onclick={() => (isFile = false)}>{$t('common.note')}</button
					>
					<button
						type="button"
						role="tab"
						data-testid="switch-file"
						aria-selected={isFile}
						class:active={isFile}
						onclick={() => (isFile = true)}>{$t('common.file')}</button
					>
				</div>
			{/if}

			<div class="body paste-indicator">
				{#if isPasting}
					<div class="pasting-overlay">
						<div class="pasting-spinner"></div>
						<span>{$t('home.pasting')}</span>
					</div>
				{/if}
				{#if isFile}
					<FileUpload data-testid="file-upload" bind:files />
				{:else}
					<TextArea
						data-testid="text-field"
						aria-label={$t('common.note')}
						bind:value={textContent}
						placeholder="Paste a password, a note or drop text here"
					/>
				{/if}

				<PastedFilesPreview bind:files />
			</div>

			<div class="toolbar">
				{#if $status?.allow_advanced}
					<Switch
						data-testid="switch-advanced"
						label={$t('common.advanced')}
						bind:value={advanced}
					/>
				{/if}
				<div class="grow"></div>
				<small class="max">{$t('common.max')} <MaxSize /></small>
				<Button type="submit">{$t('common.create')}</Button>
			</div>

			{#if advanced}
				<div class="options" transition:slide={{ duration: 200 }}>
					<AdvancedParameters bind:note bind:timeExpiration bind:customPassword />
				</div>
			{/if}
		</fieldset>
	</form>

	<p class="hint" aria-live="polite">
		{#if loading}
			{loading} <Loader />
		{:else}
			{description}
		{/if}
	</p>
{/if}

<style>
	.hero {
		margin-bottom: 1.75rem;
	}

	h1 {
		margin: 0 0 0.75rem;
		font-size: clamp(2.25rem, 7vw, 3.25rem);
		font-weight: 700;
		letter-spacing: -0.035em;
		line-height: 1;
	}

	.intro {
		max-width: 34rem;
		color: var(--ui-text-1);
		font-size: 1.0625rem;
	}

	.composer {
		background: var(--ui-surface);
		border: 1px solid var(--ui-bg-1);
		border-radius: var(--ui-radius);
		box-shadow: var(--ui-shadow);
		overflow: hidden;
		transition: var(--ui-anim);
	}

	.composer:focus-within {
		border-color: var(--ui-clr-primary);
		box-shadow: var(--ui-shadow), var(--ui-ring);
	}

	/* security-envelope tint: the one decorative element */
	.tabs {
		display: flex;
		gap: 0.25rem;
		padding: 0.625rem;
		border-bottom: 1px solid var(--ui-bg-1);
		background-color: var(--ui-field);
		background-image: repeating-linear-gradient(
			-45deg,
			var(--ui-tint) 0 1px,
			transparent 1px 7px
		);
	}

	.tabs button {
		padding: 0.375rem 0.875rem;
		border-radius: 999px;
		font-size: 0.875rem;
		font-weight: 550;
		color: var(--ui-text-1);
		cursor: pointer;
	}

	.tabs button::first-letter {
		text-transform: uppercase;
	}

	.tabs button:hover {
		color: var(--ui-text-0);
	}

	.tabs button.active {
		background: var(--ui-surface);
		color: var(--ui-text-0);
		box-shadow:
			0 0 0 1px var(--ui-bg-1),
			0 1px 2px rgb(20 27 45 / 0.08);
	}

	.body {
		padding: 0.25rem 0.25rem 0;
	}

	.body :global(.box) {
		min-height: 13rem;
		border: none;
		background: transparent;
		box-shadow: none;
		font-size: 1rem;
	}

	.body :global(textarea::placeholder) {
		color: var(--ui-text-1);
		opacity: 0.7;
	}

	.body :global(label:has(input[type='file']) .box) {
		margin: 0.75rem;
		width: calc(100% - 1.5rem);
		min-height: 11rem;
		border: 1.5px dashed var(--ui-bg-2);
		border-radius: var(--ui-radius-sm);
	}

	.toolbar {
		display: flex;
		flex-wrap: wrap;
		align-items: center;
		gap: 0.5rem 1rem;
		padding: 0.75rem 0.75rem 0.75rem 1rem;
		border-top: 1px solid var(--ui-bg-1);
	}

	.toolbar :global(small::first-letter) {
		text-transform: uppercase;
	}

	.options {
		padding: 1.25rem 1rem;
		border-top: 1px solid var(--ui-bg-1);
		background: var(--ui-field);
	}

	.hint {
		margin-top: 0.875rem;
		padding-left: 0.25rem;
		font-size: 0.875rem;
		color: var(--ui-text-1);
	}

	.hint::first-letter {
		text-transform: uppercase;
	}

	.grow {
		flex: 1;
	}

	@media screen and (max-width: 30rem) {
		.max {
			display: none;
		}
	}

	.paste-indicator {
		position: relative;
	}

	.pasting-overlay {
		position: absolute;
		inset: 0;
		background: rgba(13, 18, 32, 0.6);
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		color: white;
		z-index: 10;
	}

	.pasting-spinner {
		width: 24px;
		height: 24px;
		border: 2px solid rgba(255, 255, 255, 0.3);
		border-top: 2px solid white;
		border-radius: 50%;
		animation: spin 1s linear infinite;
		margin-bottom: 0.5rem;
	}

	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}
</style>
