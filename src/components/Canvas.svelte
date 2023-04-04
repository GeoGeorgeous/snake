<script>
    import { onMount } from "svelte";
    export let snake;
    export let food;
    export let effects = true;

    export let colors;
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
        snake.forEach((part) => {
            drawPart(part);
        });
    };

    const drawFood = () => {
        context.fillStyle = colors.fruit;
        context.fillRect(food.x, food.y, 10, 10);
    };

    const clearCanvas = () => {
        context.fillStyle = colors.bg;
        context.strokeStyle = colors.bg;
        context.fillRect(0, 0, canvas.width, canvas.height);
        context.strokeRect(0, 0, canvas.width, canvas.height);
    };

    const updateCanvas = () => {
        clearCanvas();
        drawFood();
        drawSnake();
    };

    $: if (context && snake) {
        console.log(snake);
        updateCanvas();
    }

    onMount(() => {
        context = canvas.getContext("2d");
    });
</script>

<canvas
    class:canvasEffects={effects}
    {height}
    {width}
    class="canvasBlur"
    id="gameCanvas"
    bind:this={canvas}
/>

<style>
    canvas {
        border: 4px solid var(--color-blue);
        margin-bottom: 1.5rem;
    }

    .canvasEffects {
        animation: canvasEff 3s infinite;
    }

    @keyframes canvasEff {
        0% {
            filter: blur(0.007em);
            opacity: 0.96019;
        }
        25% {
            filter: blur(0.017em);
            opacity: 0.64019;
        }

        50% {
            filter: blur(0.024em);
            opacity: 0.92019;
        }
        75% {
            filter: blur(0em);
            opacity: 0.561019;
        }

        100% {
            filter: blur(0.015em);
            opacity: 1;
        }
    }
</style>
