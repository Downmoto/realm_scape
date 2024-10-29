<script lang="ts">
	import { createEventDispatcher, type Snippet } from 'svelte';

	// Define props
	let {
		id,
		title = 'Window',
		x = $bindable(100),
		y = $bindable(100),
		w = $bindable(300),
		h = $bindable(200),
    zIndex = $bindable(),
		children
	}: {
		id: string;
		title: string;
		x: number;
		y: number;
		w: number;
		h: number;
    zIndex: number;
		children?: Snippet;
	} = $props();

  const dispatch = createEventDispatcher();

  // Dispatch focus event on click
  function handleClick() {
    dispatch('focus', { id });
  }

  // Dispatch close event on button click
  function handleClose() {
    dispatch('close', { id });
  }
  // Snap threshold (distance in pixels within which snapping occurs)
const SNAP_THRESHOLD = 15;

// Helper function to check and adjust for snapping
function checkSnap() {
  // Screen edges as snap targets
  const screenEdges = {
    left: 0,
    top: 0,
    right: window.innerWidth,
    bottom: window.innerHeight,
  };

  // Snap to screen edges
  if (Math.abs(x - screenEdges.left) < SNAP_THRESHOLD) {
    x = screenEdges.left;
  } else if (Math.abs(x + w - screenEdges.right) < SNAP_THRESHOLD) {
    x = screenEdges.right - w;
  }

  if (Math.abs(y - screenEdges.top) < SNAP_THRESHOLD) {
    y = screenEdges.top;
  } else if (Math.abs(y + h - screenEdges.bottom) < SNAP_THRESHOLD) {
    y = screenEdges.bottom - h;
  }
}

	// Internal state for dragging
	let isDragging = false;
	let startX: number, startY: number, offsetX: number, offsetY: number;

	// Internal state for resizing
	let isResizing = false;
	let startWidth: number, startHeight: number;

	// Dragging functions
	function startDrag(event: MouseEvent) {
		isDragging = true;
		startX = event.clientX;
		startY = event.clientY;
		offsetX = x;
		offsetY = y;

		window.addEventListener('mousemove', drag);
		window.addEventListener('mouseup', stopDrag);
	}

	function drag(event: MouseEvent) {
  if (isDragging) {
    // Calculate new position based on mouse movement
    x = offsetX + (event.clientX - startX);
    y = offsetY + (event.clientY - startY);
    
    // Constrain window within screen bounds
    const maxX = window.innerWidth - w - 2;  // Right boundary
    const maxY = window.innerHeight - h - 2; // Bottom boundary

    if (x < 0) x = 0;             // Left boundary
    if (y < 0) y = 0;             // Top boundary
    if (x > maxX) x = maxX;       // Right boundary
    if (y > maxY) y = maxY;       // Bottom boundary

    // Call checkSnap to adjust position if near a snap target
    checkSnap();
  }
}

	function stopDrag() {
		if (isDragging) {
			isDragging = false;
			window.removeEventListener('mousemove', drag);
			window.removeEventListener('mouseup', stopDrag);
		}
	}

	// Resizing functions
	function startResize(event: MouseEvent) {
		isResizing = true;
		startX = event.clientX;
		startY = event.clientY;
		startWidth = w;
		startHeight = h;

		window.addEventListener('mousemove', resize);
		window.addEventListener('mouseup', stopResize);
	}

	function resize(event: MouseEvent) {
		if (isResizing) {
			w = Math.max(100, startWidth + (event.clientX - startX));
			h = Math.max(100, startHeight + (event.clientY - startY));
		}
	}

	function stopResize() {
		if (isResizing) {
			isResizing = false;
			window.removeEventListener('mousemove', resize);
			window.removeEventListener('mouseup', stopResize);
		}
	}
</script>

<!-- Component structure -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<div class="window no-select" style="left: {x}px; top: {y}px; width: {w}px; height: {h}px; z-index: {zIndex}" onmousedown={handleClick}>
	<div class="title-bar no-select" onmousedown={startDrag} role="toolbar" tabindex="-1">
		{title}
    <button onclick={handleClose}>x</button>
	</div>
	<div class="content-wrapper">
		<div class="content">
			{@render children?.()}
		</div>
	</div>
	<!-- Resizable corner -->
	<div class="resize-corner" onmousedown={startResize} role="toolbar" tabindex="-1"></div>
</div>

<style>
	.window {
		position: absolute;
		border: 1px solid #ddd;
		background-color: #fff;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
		overflow: hidden;
	}

	.title-bar {
		padding: 10px;
		cursor: move;
		background-color: #333;
		color: white;
		font-weight: bold;
	}

  .content-wrapper {
    /* Ensure the content wrapper takes up the rest of the window, below the title bar */
    height: calc(100% - 40px); /* Adjust if title bar height changes */
    overflow: auto; /* Only this part scrolls */
  }

	.content {
		padding: 10px;
	}

	/* Styling for the resizable corner */
	.resize-corner {
		position: absolute;
		bottom: 0;
		right: 0;
		width: 10px;
		height: 10px;
		cursor: nwse-resize;
		background-color: transparent;
	}

	/* Prevent text selection while dragging or resizing */
	.no-select {
		user-select: none;
	}
</style>
