<script lang="ts">
	import { onMount } from 'svelte';

	onMount(() => {
		const setTheme = () => {
			if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
				document.documentElement.classList.add('dark');
			} else {
				document.documentElement.classList.remove('dark');
			}
		};

		// Set theme on initial load
		setTheme();

		// Watch for changes in the media query's value
		window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', setTheme);

		return () => {
			// Cleanup listener on component destroy
			window.matchMedia('(prefers-color-scheme: dark)').removeEventListener('change', setTheme);
		};
	});
</script>

<main>
	<slot />
</main>

<style>
	:root {
		--text: #211a12;
		--background: #f0ebe5;
		--primary: #8f9a7a;
		--secondary: #b37c35;
		--accent: #325560;
	}

	:root.dark {
		--text: #f0ebe5;
		--background: #211a12;
		--primary: #84b0e9;
		--secondary: #b2b2e1;
		--accent: #7e7e9a;
	}

	:root {
		color: var(--text);
		background-color: var(--background);
		font-family:
			system-ui,
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			Roboto,
			Oxygen,
			Ubuntu,
			Cantarell,
			'Open Sans',
			'Helvetica Neue',
			sans-serif;
		box-sizing: border-box;
		-webkit-font-smoothing: antialiased;
	}

	:global(body) {
		margin: 0;
		display: flex;
		flex-direction: column;
		min-height: 100vh;
	}

	:global(*, *:before, *:after) {
		box-sizing: inherit;
	}

	:global(:root.dark *, :root.dark *:before, :root.dark *:after) {
		color-scheme: dark;
	}

	main {
		position: relative;
		flex: 1;
		display: flex;
		flex-direction: column;
		width: 100%;
		max-height: 100vh;
		max-height: 100dvh;
		margin: 0 auto;
		overflow: hidden;
	}
</style>
