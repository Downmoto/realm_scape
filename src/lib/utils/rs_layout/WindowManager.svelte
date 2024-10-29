<script lang="ts">
	import Window from './Window.svelte';
    import { v4 as uuidv4 } from 'uuid';


	// Define a type for each window's properties
	interface WindowProps {
		id: string;
		title: string;
		x: number;
		y: number;
		w: number;
		h: number;
		zIndex: number;
	}

	// State: array of windows
	let windows: WindowProps[] = [];
	let zCounter = 1; // Track z-index for focused windows

	// Function to add a new window
	function addWindow() {
		const newWindow: WindowProps = {
			id: `${uuidv4()}`,
			title: `Window ${windows.length + 1}`,
			x: 100 + windows.length * 20, // Stagger positions
			y: 100 + windows.length * 20,
			w: 300,
			h: 200,
			zIndex: zCounter++
		};
		windows = [...windows, newWindow];
	}

	// Function to bring a window to the front
	function focusWindow(id: string) {
		const maxZIndex = Math.max(...windows.map((win) => win.zIndex), 0);
		windows = windows.map((win) => (win.id === id ? { ...win, zIndex: maxZIndex + 1 } : win));
	}

	// Function to remove a window by id
	function removeWindow(id: string) {
		windows = windows.filter((win) => win.id !== id);
	}
</script>

<div class="window-manager">
	<button on:click={addWindow}>Add Window</button>

	{#each windows as { id, title, x, y, w, h, zIndex } (id)}
		<Window
			bind:x
			bind:y
			bind:w
			bind:h
			{id}
			{title}
			{zIndex}
			on:focus={(e) => focusWindow(e.detail.id)}
			on:close={(e) => removeWindow(e.detail.id)}
		/>
	{/each}
</div>

<style>
	.window-manager {
		position: relative;
		width: 100vw;
		height: 100vh;
	}
</style>
