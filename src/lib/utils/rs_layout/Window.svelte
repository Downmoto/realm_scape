<script lang="ts">
	import type { Snippet } from "svelte";

  // Define props
  let {
    id,
    title = "Window",
    x = 100,
    y = 100,
    w = 300,
    h = 200,
    children
  } : {
    id: string;
    title: string;
    x: number;
    y: number;
    w: number;
    h: number;
    children?: Snippet
  } = $props();

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
      x = offsetX + (event.clientX - startX);
      y = offsetY + (event.clientY - startY);
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
<div class="window no-select" style="left: {x}px; top: {y}px; width: {w}px; height: {h}px;">
  <div class="title-bar no-select" onmousedown={startDrag} role="toolbar" tabindex="-1">
    {title}
  </div>
  <div class="content">
    {@render children?.()}
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
  }

  .title-bar {
    padding: 10px;
    cursor: move;
    background-color: #333;
    color: white;
    font-weight: bold;
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
