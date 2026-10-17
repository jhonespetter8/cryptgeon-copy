<script lang="ts">
	import { status } from '$lib/stores/status'
	import AboutParagraph from '$lib/ui/AboutParagraph.svelte'
</script>

<svelte:head>
	<title>About</title>
</svelte:head>

<section class="content">
	<h1>About</h1>

	<p>Share passwords, notes and files securely through a one-time link.</p>

	<AboutParagraph title="How it works">
		<span>
			Content is encrypted in your browser (AES-GCM) before it is sent. The key stays in the link and
			never reaches the server. Data is kept in memory only and deleted after the view or time limit.
		</span>
	</AboutParagraph>

	{#if $status}
		<p class="version"><code>v{$status.version}</code></p>
	{/if}
</section>

<style>
	section {
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	h1 {
		margin: 0;
	}

	.version {
		opacity: 0.6;
	}
</style>
