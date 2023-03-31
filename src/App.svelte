<script>
  // @ts-nocheck

  import { onMount } from "svelte";
  import Canvas from "./components/Canvas.svelte";

  let canvas;

  let gameConfig = {
    width: 400, // canvas width
    height: 420, // canvas height
    fps: 10, // frames per second (canvas update)
    colors: {
      bg: "#112222",
      main: "#203e4a",
      light: "#4e94b0",
    },
  };

  let gameState = {
    snake: [
      { x: 200, y: 210 },
      { x: 190, y: 210 },
      { x: 180, y: 210 },
      { x: 170, y: 210 },
      { x: 160, y: 210 },
    ],
  };

  let score = 0;

  let dx = 10;
  let dy = 0;

  let food_x;
  let food_y;

  let changinDirecion = false;

  function change_direction(event) {
    if (changinDirecion) return;
    changinDirecion = true;

    const LEFT_KEY = 37;
    const RIGHT_KEY = 39;
    const UP_KEY = 38;
    const DOWN_KEY = 40;

    const keyPressed = event.keyCode;
    const goingUp = dy === -10;
    const goingDown = dy === 10;
    const goingRight = dx === 10;
    const goingLeft = dx === -10;

    if (keyPressed === LEFT_KEY && !goingRight) {
      dx = -10;
      dy = 0;
    }

    if (keyPressed === UP_KEY && !goingDown) {
      dx = 0;
      dy = -10;
    }

    if (keyPressed === RIGHT_KEY && !goingLeft) {
      dx = 10;
      dy = 0;
    }

    if (keyPressed === DOWN_KEY && !goingUp) {
      dx = 0;
      dy = 10;
    }
  }

  // function moveSnake() {
  //   // Create the new Snake's head
  //   const head = { x: snake[0].x + dx, y: snake[0].y + dy };
  //   // Add the new head to the beginning of snake body
  //   snake.unshift(head);
  //   const has_eaten_food = snake[0].x === food_x && snake[0].y === food_y;
  //   if (has_eaten_food) {
  //     // Increase score
  //     score = score + 10;
  //     // Generate new food location
  //   } else {
  //     // Remove the last part of snake body
  //     snake.pop();
  //   }
  // }

  const moveSnake = () => {
    const head = {
      x: gameState.snake[0].x + dx,
      y: gameState.snake[0].y + dy,
    };
    gameState.snake = [head, ...gameState.snake.slice(0, -1)];
  };

  function hasGameEnded() {
    for (let i = 4; i < snake.length; i++) {
      if (snake[i].x === snake[0].x && snake[i].y === snake[0].y) return true;
    }
    const hitLeftWall = snake[0].x < 0;
    const hitRightWall = snake[0].x > canvas.width - 10;
    const hitToptWall = snake[0].y < 0;
    const hitBottomWall = snake[0].y > canvas.height - 10;
    return hitLeftWall || hitRightWall || hitToptWall || hitBottomWall;
  }

  function random_food(min, max) {
    return Math.round((Math.random() * (max - min) + min) / 10) * 10;
  }

  function gen_food() {
    food_x = random_food(0, canvas.width - 10);
    food_y = random_food(0, canvas.height - 10);
    snake.forEach(function has_snake_eaten_food(part) {
      const has_eaten = part.x == food_x && part.y == food_y;
      if (has_eaten) gen_food();
    });
  }

  function drawFood() {
    context.fillStyle = "lightgreen";
    context.strokestyle = "darkgreen";
    context.fillRect(food_x, food_y, 10, 10);
    context.strokeRect(food_x, food_y, 10, 10);
  }

  // const run = (canvas, context) => {
  //   if (hasGameEnded()) return;

  //   changinDirecion = false;
  //   setTimeout(() => {
  //     clearCanvas(canvas, context);
  //     drawFood();
  //     moveSnake();
  //     drawSnake();
  //     run(canvas, context);
  //   }, gameSpeed);
  // };

  const run = () => {
    let interval = setInterval(() => {
      moveSnake();
    }, 1000 / gameConfig.fps);

    // clearInterval(interval)
  };

  onMount(() => {
    // document.addEventListener("keydown", change_direction);
    run();
    // gen_food();
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
      {gameState}
    />
    <button class="replay-button" type="button">Play again</button>
    <p class="controls">
      Move with <span class="key">w</span>
      <span class="key">a</span>
      <span class="key">s</span>
      <span class="key">d</span> or <span class="key">arrow keys</span>
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
    font-size: 1.5rem;
  }

  .replay-button {
    margin: 1.5rem 0;
    background-color: var(--color-blue);
    border: none;
    font-size: 1.5rem;
    color: var(--color-light);
    padding: 1rem;
  }

  .replay-button:hover,
  .replay-button:focus {
    outline: 2px solid var(--color-light);
    outline-offset: -6px;
  }

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
  }

  .controls {
    text-align: center;
    font-size: 1.2rem;
  }
</style>
