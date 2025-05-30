<script>
    import { createEventDispatcher, onMount } from 'svelte';
    import { cubicOut } from 'svelte/easing';
    import { tweened } from 'svelte/motion';

    const MIN_HEIGHT = 300;    // collapsed height (px)
    const MAX_HEIGHT = 400;   // expanded height (px)

    const dispatch = createEventDispatcher();

    // tweened store for the sheet’s current height
    const height = tweened(MIN_HEIGHT, { duration: 200, easing: cubicOut });

    let startY;
    let startHeight;
    let dragging = false;

    function onPointerDown(e) {
        dragging = true;
        startY = e.clientY;
        height.subscribe(h => startHeight = h)();
        window.addEventListener('pointermove', onPointerMove);
        window.addEventListener('pointerup', onPointerUp);
    }

    function onPointerMove(e) {
        if (!dragging) return;
        const dy = startY - e.clientY;
        let newH = startHeight + dy;
        newH = Math.min(Math.max(newH, MIN_HEIGHT), MAX_HEIGHT);
        height.set(newH);
    }

    function onPointerUp() {
        dragging = false;
        window.removeEventListener('pointermove', onPointerMove);
        window.removeEventListener('pointerup', onPointerUp);

        // snap to nearest state
        height.subscribe(h => {
            const mid = (MIN_HEIGHT + MAX_HEIGHT) / 2;
            const target = h > mid ? MAX_HEIGHT : MIN_HEIGHT;
            height.set(target);
            dispatch('stateChange', { expanded: target === MAX_HEIGHT });
        })();
    }
</script>

<style>
    .sheet {
        position: fixed;
        left: 0;
        bottom: 0;
        width: 100%;
        z-index: 3000;
        background: white;
        box-shadow: 0 -4px 12px rgba(0,0,0,0.1);
        border-top-left-radius: 16px;
        border-top-right-radius: 16px;
        touch-action: none;
        display: flex;
        flex-direction: column;
    }

    .handle {
        width: 40px;
        height: 4px;
        background: #ccc;
        border-radius: 2px;
        margin: 8px auto;
        cursor: grab;
    }

    .content {
        overflow: auto;
        padding: 16px;
        flex: 1;
    }
</style>

<div
  class="sheet"
  style="height: {$height}px; transform: translateY({MAX_HEIGHT - $height}px)"
  on:pointerdown={onPointerDown}
>
    <div class="handle"></div>
    <div class="content">
        <slot />
    </div>
</div>