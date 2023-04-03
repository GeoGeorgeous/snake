<script>
  import { onMount } from "svelte";

  // @ts-nocheck

  import Canvas from "./components/Canvas.svelte";
  import Controls from "./components/Controls.svelte";
  import { gameConfig } from "./lib/cfg";

  let canvas;
  let hasGameEnded = false;
  let runtime;

  let snake = [
    { x: 200, y: 210 },
    { x: 190, y: 210 },
    { x: 180, y: 210 },
    { x: 170, y: 210 },
    { x: 160, y: 210 },
    { x: 150, y: 210 },
    { x: 140, y: 210 },
    { x: 130, y: 210 },
  ];

  let food = {};

  let direction = "right";
  let directionChange = false;
  let score = 0;
  let dx;
  let dy;

  const saveHighStore = (score) => {
    localStorage.setItem("snakeHighScore", score);
  };

  $: highScore = score > highScore ? score : highScore || 0;
  $: if (highScore !== 0) saveHighStore(highScore);
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
      x: snake[0].x + dx,
      y: snake[0].y + dy,
    };
    snake = [head, ...snake.slice(0, -1)];
    const hasEaten = snake[0].x === food.x && snake[0].y === food.y;
    if (hasEaten) {
      // growing the snake
      snake = [head, ...snake];
      score = score + 1;
      generateFood();
    }
  };

  const randomFood = (min, max) => {
    return Math.round((Math.random() * (max - min) + min) / 10) * 10;
  };

  const generateFood = () => {
    food.x = randomFood(0, gameConfig.width - 10);
    food.y = randomFood(0, gameConfig.height - 10);
    snake.forEach((part) => {
      const foodCollapse = part.x == food.x && part.y == food.y;
      if (foodCollapse) generateFood();
    });
  };

  const reset = () => {
    clearTimeout(runtime);
    // Reset all the game states
    snake = [
      { x: 200, y: 210 },
      { x: 190, y: 210 },
      { x: 180, y: 210 },
      { x: 170, y: 210 },
      { x: 160, y: 210 },
      { x: 170, y: 210 },
      { x: 180, y: 210 },
      { x: 190, y: 210 },
    ];
    food = {};
    directionChange = false;
    direction = "right";
    score = 0;
    run();
  };

  const checkGameStatus = () => {
    let selfEaten = snake
      .slice(1)
      .some((part) => part.x === snake[0].x && part.y === snake[0].y);

    const hitLeftWall = snake[0].x <= 0;
    const hitRightWall = snake[0].x + 10 >= gameConfig.width;
    const hitTopWall = snake[0].y <= 0;
    const hitBottomWall = snake[0].y + 10 >= gameConfig.height;
    hasGameEnded =
      selfEaten || hitLeftWall || hitRightWall || hitTopWall || hitBottomWall;
  };

  const run = () => {
    checkGameStatus();
    if (hasGameEnded) return;
    runtime = setTimeout(() => {
      moveSnake();
      directionChange = false;
      run();
    }, 1000 / gameConfig.fps);
  };

  onMount(() => {
    const storedScore = localStorage.getItem("snakeHighScore");
    if (storedScore) {
      highScore = storedScore;
    }

    generateFood();
    run();
  });
</script>

<main class="main">
  <div class="wrapper">
    <div class="score">
      <p>Score: {score}</p>

      <p>High Score: {highScore}</p>
    </div>
    <Canvas
      bind:canvas
      width={gameConfig.width}
      height={gameConfig.height}
      colors={gameConfig.colors}
      {hasGameEnded}
      {snake}
      {food}
    />
    <Controls on:directionChange={handleDirectionChange} on:restart={reset} />
    <div class="controls" />
    <p class="controls__text">
      Move with <span class="key">W</span>
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
    font-family: monospace;
    color: var(--color-light);
    text-align: center;
    font-size: 1.5rem;
  }

  .controls__text {
    font-size: 1.2rem;
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
    font-size: 1.2rem;
    font-weight: bold;
  }

  .controls {
    display: flex;
    gap: 1rem;
  }
</style>
