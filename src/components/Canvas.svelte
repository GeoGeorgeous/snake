<script>
    import { onMount } from "svelte";

    export let colors;
    export let gameState;
    export let width;
    export let height;
    export let canvas;
    let context;

    const drawPart = (part) => {
        context.fillStyle = colors.light;
        context.strokeStyle = colors.main;
        context.fillRect(part.x, part.y, 10, 10);
        context.strokeRect(part.x, part.y, 10, 10);
    };

    const drawSnake = () => {
        gameState.snake.forEach((part) => {
            drawPart(part);
        });
    };

    const clearCanvas = () => {
        context.fillStyle = colors.bg;
        context.strokeStyle = colors.bg;
        context.fillRect(0, 0, canvas.width, canvas.height);
        context.strokeRect(0, 0, canvas.width, canvas.height);
    };

    const updateCanvas = () => {
        clearCanvas(); // Clear frame
        drawSnake(); // Draw new frame
    };

    $: if (context && gameState) {
        updateCanvas();
    }

    onMount(() => {
        context = canvas.getContext("2d");
        updateCanvas();
    });
</script>

<canvas {height} {width} id="gameCanvas" bind:this={canvas} />

<style>
    canvas {
        border: 4px solid var(--color-blue);
    }
</style>
