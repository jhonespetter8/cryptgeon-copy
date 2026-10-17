<script lang="ts" module>
	import { writable } from 'svelte/store'

	const themes = ['dark', 'light', 'auto'] as const
	type Theme = (typeof themes)[number]

	const NextTheme: Record<Theme, Theme> = {
		auto: 'light',
		light: 'dark',
		dark: 'auto',
	}

	function init(): Theme {
		if (typeof window !== 'undefined') {
			const saved = window.localStorage.getItem('theme') as Theme
			if (themes.includes(saved)) return saved
		}
		return 'auto'
	}

	export const theme = writable<Theme>(init())

	theme.subscribe((theme) => {
		if (typeof window !== 'undefined') {
			window.localStorage.setItem('theme', theme)
			const html = window.document.getElementsByTagName('html')[0]
			html.setAttribute('theme', theme)
		}
	})
</script>

<script lang="ts">
	import Icon from '$lib/ui/Icon.svelte'

	function change() {
		theme.update((current) => NextTheme[current])
	}
</script>

<button onclick={change}>
	<Icon class="icon" icon="contrast" />
	{$theme}
</button>

<style>
	button :global(.icon) {
		height: 0.9rem;
		width: 0.9rem;
	}

	button {
		display: inline-flex;
		align-items: center;
		gap: 0.4rem;
		padding: 0.35rem 0.75rem;
		border: 1px solid var(--ui-bg-1);
		border-radius: 999px;
		font-size: 0.8125rem;
		color: var(--ui-text-1);
		text-transform: capitalize;
		cursor: pointer;
	}

	button:hover {
		color: var(--ui-text-0);
		border-color: var(--ui-bg-2);
	}
</style>
