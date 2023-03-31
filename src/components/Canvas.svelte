<script>
    import { onMount } from "svelte";

    export let colors;
    export let game;
    export let width;
    export let height;
    export let canvas;
    export let hasGameEnded;
    let context;

    const drawPart = (part) => {
        context.fillStyle = hasGameEnded ? "red" : colors.light;
        context.strokeStyle = colors.main;
        context.fillRect(part.x, part.y, 10, 10);
        context.strokeRect(part.x, part.y, 10, 10);
    };

    const drawSnake = () => {
        game.snake.forEach((part) => {
            drawPart(part);
        });
    };

    const drawFood = () => {
        console.log(game.food.x, game.food.y);
        context.fillStyle = colors.fruit;
        context.fillRect(game.food.x, game.food.x, 10, 10);
    };

    const clearCanvas = () => {
        context.fillStyle = colors.bg;
        context.strokeStyle = colors.bg;
        context.fillRect(0, 0, canvas.width, canvas.height);
        context.strokeRect(0, 0, canvas.width, canvas.height);
    };

    const updateCanvas = () => {
        clearCanvas();
        if (game.food) drawFood();
        drawSnake();
    };

    $: if (context && game) {
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
        margin-bottom: 1.5rem;
    }
</style>
