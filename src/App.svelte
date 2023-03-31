<script>
  import { onMount } from "svelte";

  // @ts-nocheck

  import Canvas from "./components/Canvas.svelte";
  import Controls from "./components/Controls.svelte";
  import { gameConfig } from "./lib/cfg";

  let canvas;

  let game = {
    snake: [
      { x: 200, y: 210 },
      { x: 190, y: 210 },
      { x: 180, y: 210 },
      { x: 170, y: 210 },
      { x: 160, y: 210 },
    ],
  };
  let direction = "right";
  let directionChange = false;
  let score = 0;
  let dx;
  let dy;

  const handleDirectionChange = (event) => {
    if (directionChange) return;

    directionChange = true;
    switch (event.detail) {
      case "up":
        if (direction !== "down") {
          direction = "up";
          dx = 0;
          dy = -10;
        }
        break;
      case "right":
        if (direction !== "left") {
          direction = "right";
          dx = 10;
          dy = 0;
        }
        break;
      case "down":
        if (direction !== "up") {
          direction = "down";
          dx = 0;
          dy = 10;
        }
        break;
      case "left":
        if (direction !== "right") {
          direction = "left";
          dx = -10;
          dy = 0;
        }
        break;
    }
  };

  const moveSnake = () => {
    const head = {
      x: game.snake[0].x + dx,
      y: game.snake[0].y + dy,
    };
    game.snake = [head, ...game.snake.slice(0, -1)];
  };

  const reset = () => {
    // Reset all the game states
    game.snake = [
      { x: 200, y: 210 },
      { x: 190, y: 210 },
      { x: 180, y: 210 },
      { x: 170, y: 210 },
      { x: 160, y: 210 },
    ];
    directionChange = false;
    direction = "right";
    score = 0;
  };

  $: {
    if (direction === "left") {
      dx = -10;
      dy = 0;
    }

    if (direction === "up") {
      dx = 0;
      dy = -10;
    }

    if (direction === "right") {
      dx = 10;
      dy = 0;
    }

    if (direction === "down") {
      dx = 0;
      dy = 10;
    }
  }

  const isGameEnded = false;

  const run = () => {
    if (isGameEnded) return;
    setTimeout(() => {
      moveSnake();
      directionChange = false;
      run();
    }, 1000 / gameConfig.fps);
  };

  onMount(() => {
    run();
  });
</script>

<main class="main">
  <div class="wrapper">
    <div class="score">
      <p>Score: {score}</p>

      <p>High Score: 0</p>
    </div>
    <Canvas
      bind:canvas
      width={gameConfig.width}
      height={gameConfig.height}
      colors={gameConfig.colors}
      {game}
    />
    <Controls on:directionChange={handleDirectionChange} on:restart={reset} />
    <div class="controls" />
    <p class="controls__text">
      Move with <span class="key">w</span>
      <span class="key">A</span>
      <span class="key">S</span>
      <span class="key">D</span> or <span class="key">ARROW KEYS</span>
    </p>
    <p class="controls__text">
      Restart with <span class="key">R</span>
    </p>
  </div>
</main>

<style>
  :root {
    --color-green: #112222;
    --color-blue: #203e4a;
    --color-light: #4e94b0;
    font-family: monospace;
    line-height: 1rem;
  }

  .main {
    background-color: var(--color-green);
    height: 100vh;
    display: grid;
    place-items: center center;
  }

  p {
    color: var(--color-light);
    text-align: center;
    font-size: 1.5rem;
  }

  .controls__text {
    font-size: 1.1rem;
    margin-bottom: 1.5rem;
  }

  .controls__text:last-of-type {
    margin: 0;
  }

  /* button {
    margin: 1.5rem 0;
    background-color: var(--color-blue);
    border: none;
    font-size: 1.5rem;
    color: var(--color-light);
    padding: 1rem;
  }

  button:hover,
  button:focus {
    outline: 2px solid var(--color-light);
    outline-offset: -6px;
  } */

  .wrapper {
    display: flex;
    flex-flow: column nowrap;
    position: relative;
  }

  .score {
    display: flex;
    flex-flow: row nowrap;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  .key {
    background-color: var(--color-blue);
    outline: 1px solid var(--color-light);
    outline-offset: -4px;
    padding: 5px 10px;
    font-size: 1rem;
  }

  .controls {
    display: flex;
    gap: 1rem;
  }
</style>
