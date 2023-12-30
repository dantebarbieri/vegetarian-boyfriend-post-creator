<script lang="ts">
	import { onMount } from 'svelte';
	import { quintInOut } from 'svelte/easing';
	import { writable } from 'svelte/store';
	import { fly } from 'svelte/transition';

	import Card from './Card.svelte';
	import Post from './Post.svelte';

	function createLocalDate(dateString: string) {
		const [year, month, day] = dateString.split('-').map(Number);
		return new Date(year, month - 1, day);
	}

	// Function to save to local storage
	function saveToLocalStorage(key: string, value: any) {
		if (typeof window !== 'undefined') {
			localStorage.setItem(key, JSON.stringify(value));
		}
	}

	// Function to get from local storage
	function getFromLocalStorage(key: string) {
		if (typeof window !== 'undefined') {
			const value = localStorage.getItem(key);
			return value ? JSON.parse(value) : '';
		}
		return '';
	}

	// Create writable stores for your inputs with no initial values
	const categoryStore = writable<string>();
	const slugStore = writable<string>();
	const titleStore = writable<string>();
	const rawDateStore = writable<string>();
	const descriptionStore = writable<string>();
	const contentStore = writable<string>();
	const imgSourceFallbackStore = writable<string>();
	const imgSourceWebpStore = writable<string>();

	let isInitialized = false;

	// Initialize stores from local storage when the component mounts
	onMount(() => {
		categoryStore.set(getFromLocalStorage('category'));
		slugStore.set(getFromLocalStorage('slug'));
		titleStore.set(getFromLocalStorage('title'));
		rawDateStore.set(getFromLocalStorage('rawDate'));
		descriptionStore.set(getFromLocalStorage('description'));
		contentStore.set(getFromLocalStorage('content'));
		imgSourceFallbackStore.set(getFromLocalStorage('imgSourceFallback'));
		imgSourceWebpStore.set(getFromLocalStorage('imgSourceWebp'));
		isInitialized = true;
	});

	// Update local storage only after initial values are set
	function updateLocalStorage() {
		if (!isInitialized) return;

		saveToLocalStorage('category', $categoryStore);
		saveToLocalStorage('slug', $slugStore);
		saveToLocalStorage('title', $titleStore);
		saveToLocalStorage('rawDate', $rawDateStore);
		saveToLocalStorage('description', $descriptionStore);
		saveToLocalStorage('content', $contentStore);
		saveToLocalStorage('imgSourceFallback', $imgSourceFallbackStore);
		saveToLocalStorage('imgSourceWebp', $imgSourceWebpStore);
	}

	let toast: HTMLDivElement;
	let toastMessage: string;
	let toastTimeoutId: number;

	function showToast(message: string, duration: number = 10000) {
		clearTimeout(toastTimeoutId);
		toastMessage = message;
		toastTimeoutId = setTimeout(() => (toastMessage = ''), duration);
	}

	const handleSubmit = async (event: Event) => {
		event.preventDefault();

		// Serialize form data
		const formData = {
			category: $categoryStore,
			slug: $slugStore,
			title: $titleStore,
			date: {
				$date: date?.getTime()
			},
			description: $descriptionStore,
			content: $contentStore,
			imgSourceFallback: $imgSourceFallbackStore,
			imgSourceWebp: $imgSourceWebpStore
		};

		// Convert formData to JSON and create a Blob
		const blob = new Blob([JSON.stringify(formData)], { type: 'application/json' });
		const url = URL.createObjectURL(blob);

		// Show toast notification
		let fileName = `${$categoryStore}-${$slugStore}.json`;
		showToast(`${fileName} downloaded.`, 10000);

		// Create a link and start the download
		const a = document.createElement('a');
		a.href = url;
		a.download = fileName;
		a.click();
		URL.revokeObjectURL(url);
	};

	// Auto-subscribe to store changes and save them
	$: categoryStore.subscribe(updateLocalStorage);
	$: slugStore.subscribe(updateLocalStorage);
	$: titleStore.subscribe(updateLocalStorage);
	$: rawDateStore.subscribe(updateLocalStorage);
	$: descriptionStore.subscribe(updateLocalStorage);
	$: contentStore.subscribe(updateLocalStorage);
	$: imgSourceFallbackStore.subscribe(updateLocalStorage);
	$: imgSourceWebpStore.subscribe(updateLocalStorage);

	$: date = $rawDateStore ? createLocalDate($rawDateStore) : null;
</script>

<h1>Blog Post Creator</h1>
<p>
	Create a blog post for <a href="https://thevegetarianboyfriend.com">thevegetarianboyfriend.com</a
	>.
</p>
{#if toastMessage}
	<div
		bind:this={toast}
		transition:fly={{ duration: 1000, y: '-100%', easing: quintInOut }}
		class="toast"
	>
		{toastMessage}
	</div>
{/if}
<div class="creation-wrapper">
	<div class="form">
		<h2>Creation Form</h2>
		<form on:submit={handleSubmit}>
			<div class="form-row">
				<label for="category">Category<span class="required">*</span></label>
				<input
					id="category"
					type="text"
					name="category"
					placeholder="Category"
					required
					bind:value={$categoryStore}
				/>
			</div>
			<div class="form-row">
				<label for="slug">Slug<span class="required">*</span></label>
				<input
					id="slug"
					type="text"
					name="slug"
					placeholder="Slug"
					required
					bind:value={$slugStore}
				/>
			</div>
			<div class="form-row">
				<label for="title">Title<span class="required">*</span></label>
				<input
					id="title"
					type="text"
					name="title"
					placeholder="Title"
					required
					bind:value={$titleStore}
				/>
			</div>
			<div class="form-row">
				<label for="date">Date<span class="required">*</span></label>
				<input id="date" type="date" name="date" required bind:value={$rawDateStore} />
			</div>
			<div class="form-row">
				<label for="description">Description<span class="required">*</span></label>
				<textarea
					id="description"
					name="description"
					placeholder="Description"
					required
					bind:value={$descriptionStore}
				/>
			</div>
			<div class="form-row">
				<label for="content">Content<span class="required">*</span></label>
				<textarea
					id="content"
					name="content"
					placeholder="Content"
					required
					bind:value={$contentStore}
				/>
			</div>
			<div class="form-row">
				<label for="imgSourceFallback">Image Source Fallback<span class="required">*</span></label>
				<input
					id="imgSourceFallback"
					type="text"
					name="imgSourceFallback"
					placeholder="Image Source Fallback"
					required
					bind:value={$imgSourceFallbackStore}
				/>
			</div>
			<div class="form-row">
				<label for="imgSourceWebp">Image Source Webp</label>
				<input
					id="imgSourceWebp"
					type="text"
					name="imgSourceWebp"
					placeholder="Image Source Webp"
					bind:value={$imgSourceWebpStore}
				/>
			</div>
			<button type="submit">Submit</button>
		</form>
	</div>
	<div class="preview">
		<h2>Preview</h2>
		<div>
			<h3>Card</h3>
			{#if $slugStore && $titleStore && date && $descriptionStore && $imgSourceFallbackStore}
				<div class="border">
					<Card
						title={$titleStore}
						{date}
						description={$descriptionStore}
						imageSrcFallback={$imgSourceFallbackStore}
						link={`https://thevegetarianboyfriend.com/blog/${slugStore}`}
						imageSrcWebp={$imgSourceWebpStore}
					/>
				</div>
			{:else}
				<p>Fill out the form to see a preview.</p>
				<p>Missing:</p>
				<ul>
					{#if !$slugStore}
						<li>Slug</li>
					{/if}
					{#if !$titleStore}
						<li>Title</li>
					{/if}
					{#if !$rawDateStore}
						<li>Date</li>
					{/if}
					{#if !$descriptionStore}
						<li>Description</li>
					{/if}
					{#if !$imgSourceFallbackStore}
						<li>Image Source Fallback</li>
					{/if}
				</ul>
			{/if}
		</div>
		<div>
			<h3>Post</h3>
			{#if $titleStore && date && $contentStore && $imgSourceFallbackStore}
				<div class="border">
					<Post
						title={$titleStore}
						{date}
						content={$contentStore}
						imageSrcFallback={$imgSourceFallbackStore}
						imageSrcWebp={$imgSourceWebpStore}
					/>
				</div>
			{:else}
				<p>Fill out the form to see a preview.</p>
				<p>Missing:</p>
				<ul>
					{#if !$titleStore}
						<li>Title</li>
					{/if}
					{#if !$rawDateStore}
						<li>Date</li>
					{/if}
					{#if !$contentStore}
						<li>Content</li>
					{/if}
					{#if !$imgSourceFallbackStore}
						<li>Image Source Fallback</li>
					{/if}
				</ul>
			{/if}
		</div>
	</div>
</div>

<style>
	.creation-wrapper {
		position: relative;
		display: flex;
		gap: 1rem;
		flex: 1;
		overflow-y: scroll;
	}

	a {
		color: #0070f3;
		text-decoration: none;
	}

	.toast {
		position: absolute;
		top: 0.5rem;
		right: 0.5rem;
		padding: 1rem;
		background-color: #0070f3;
		color: white;
		border-radius: 10px;
		box-shadow: -5px 5px 5px rgba(0, 0, 0, 0.25);

		transition:
			opacity ease-in-out 0.5s,
			box-shadow ease-in-out 0.2s;
	}

	.toast:hover {
		box-shadow: -7.5px 7.5px 10px rgba(0, 0, 0, 0.5);
	}

	button {
		padding: 10px;
		border-radius: 5px;
		border: none;
		background-color: #0070f3;
		color: white;
		font-weight: bold;
		cursor: pointer;
		align-self: start;
	}

	.required {
		color: red; /* Red color for the asterisk */
		vertical-align: top; /* Positions the asterisk as superscript */
		font-size: smaller; /* Optionally, make the asterisk smaller than the label text */
		user-select: none; /* Prevents text selection */
		pointer-events: none; /* Disables cursor events */
	}

	input {
		font-family: ui-monospace, monospace;
	}

	/* Style for the form elements */
	form {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	/* Style for label */
	label {
		display: inline-block;
	}

	/* Style for input and textarea */
	input,
	textarea {
		border-radius: 3px;
		padding: 5px;
	}

	/* Ensures labels are aligned to the left of inputs */
	.form-row {
		display: flex;
		align-items: center;
		gap: 10px;
	}

	@media screen and (max-width: 600px) {
		/* On smaller screens, stack labels and inputs */
		.form-row {
			flex-direction: column;
		}
	}

	.form {
		position: sticky;
		top: 0;
		overflow-y: auto;
		padding-bottom: 2rem;
	}

	.preview {
		flex: 1;
	}

	h3 {
		margin-bottom: 0.25rem;
	}

	.border {
		width: 100%;
		height: 100%;
		border: 1px solid black;
		border-radius: 10px;
		padding: 1rem;
	}
</style>
