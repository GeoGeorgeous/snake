<script>
  import { onDestroy, onMount } from "svelte";

  // @ts-nocheck

  import Canvas from "./components/Canvas.svelte";
  import Controls from "./components/Controls.svelte";
  import { gameConfig } from "./lib/cfg";

  let canvas;
  let hasGameEnded = false;
  let runtime;
  let effects = true;
  let controlsShown = false;

  let snake = [
    { x: 200, y: 200 },
    { x: 190, y: 200 },
    { x: 180, y: 200 },
    { x: 170, y: 200 },
  ];

  let food = {};

  let mode = 0;
  let fps = 25; // frames per second (canvas update)
  let multiplier = 1;

  let direction = "right";
  let directionChange = false;
  let score = 0;
  let dx;
  let dy;

  const saveHighStore = (score) => {
    localStorage.setItem("snakeHighScore", score);
  };

  const handleSpeedChange = () => {
    const difficulty = [
      { speed: 6, multiplier: 0.25 },
      { speed: 12, multiplier: 0.5 },
      { speed: 18, multiplier: 0.75 },
      { speed: 24, multiplier: 1 },
      { speed: 32, multiplier: 1.25 },
      { speed: 60, multiplier: 2 },
    ];

    fps = difficulty[mode].speed;
    multiplier = difficulty[mode].multiplier;
    if (mode >= difficulty.length - 1) mode = 0;
    else mode = mode + 1;
    reset();
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
    snake.unshift(head);
    const hasEaten = snake[0].x === food.x && snake[0].y === food.y;
    if (hasEaten) {
      score = Math.round(score + 10 * multiplier);
      generateFood();
    } else {
      snake.pop();
    }
    snake = snake;
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
    ];
    food = {};
    direction = "right";
    directionChange = false;
    score = 0;
    generateFood();
    run();
  };

  const checkGameStatus = () => {
    let selfEaten = snake
      .slice(1)
      .some((part) => part.x === snake[0].x && part.y === snake[0].y);

    const hitLeftWall = snake[0].x < 0;
    const hitRightWall = snake[0].x + 10 > gameConfig.width;
    const hitTopWall = snake[0].y < 0;
    const hitBottomWall = snake[0].y + 10 > gameConfig.height;
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
    }, 1000 / fps);
  };

  onMount(() => {
    const storedScore = localStorage.getItem("snakeHighScore");
    if (storedScore) {
      highScore = storedScore;
    }

    generateFood();
    run();
  });

  onDestroy(() => clearTimeout(runtime));
</script>

<main class="main" class:effects>
  <div class="wrapper">
    <div class="score">
      <p>Score: <span class:colored={score > 0}>{score}</span></p>
      <p>High Score: {highScore}</p>
    </div>
    <div class="canvas-wrapper">
      <Canvas
        bind:canvas
        width={gameConfig.width}
        height={gameConfig.height}
        colors={gameConfig.colors}
        {effects}
        {hasGameEnded}
        {snake}
        {food}
      />
      <Controls
        on:directionChange={handleDirectionChange}
        on:restart={reset}
        on:toggleEffects={() => (effects = !effects)}
        on:speedChange={handleSpeedChange}
        on:toggleControls={() => (controlsShown = !controlsShown)}
      />

      {#if controlsShown}
        <div class="controls">
          <ul>
            <li>
              <span class="key">W</span> <span class="key">&#8593;</span> Move up
            </li>
            <li>
              <span class="key">A</span> <span class="key">&#8594;</span> Move right
            </li>
            <li>
              <span class="key">S</span> <span class="key">&#8595;</span> Move down
            </li>
            <li>
              <span class="key">D</span> <span class="key">&#8592;</span> Move left
            </li>
            <li>
              <span class="key">H</span> Hide Controls
            </li>
          </ul>
          <ul>
            <li>
              <span class="key">E</span> Effects
              <span class:colored={effects}>[On]</span>
              <span class:colored={!effects}>[Off]</span>
            </li>
            <li>
              <span class="key">R</span> Restart
            </li>
            <li>
              <span class="key">T</span> Toggle difficulty
            </li>
            <li>
              Speed:
              <span class="colored">{fps.toString().padStart(2, "0")}</span><br
              />
              Modifier:
              <span class="colored">{multiplier.toFixed(2)}</span>
            </li>
          </ul>
        </div>
      {:else}
        <div class="controls">
          <ul>
            <li>
              <span class="key">H</span> Show controls
            </li>
          </ul>
        </div>
      {/if}
    </div>
  </div>
</main>

<style>
  :root {
    --color-green: #112222;
    --color-blue: #203e4a;
    --color-light: #4e94b0;
    --color-accent: #90ee90;
    font-family: "VT323", monospace;
    line-height: 1rem;
    filter: blur(0.007em);
    color: var(--color-blue);
  }

  .main {
    background-color: var(--color-green);
    height: 100vh;
    display: grid;
    place-items: center center;
  }

  .effects.main {
    animation: bg 5ms infinite;
  }

  .effects.main::after {
    box-shadow: inset 0 0 10em rgba(0, 0, 0, 0.75);
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 2;
    content: "";
  }

  li {
    margin-bottom: 1rem;
  }

  p {
    font-family: "VT323", monospace;
    color: var(--color-light);
    text-align: center;
    font-size: 1.5rem;
  }

  .effects p {
    animation: blur 1s infinite alternate;
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
    font-size: 1.1rem;
  }

  .colored {
    color: var(--color-accent);
  }

  .canvas-wrapper {
    position: relative;
  }

  .controls {
    position: absolute;
    display: grid;
    gap: 0;
    grid-template-columns: 1fr 1fr;
    font-size: 1rem;
    font-family: monospace;
    color: var(--color-light);
    justify-content: space-between;
  }

  /* crt */
  .effects::before {
    content: " ";
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    background: linear-gradient(
        rgba(18, 16, 16, 0) 50%,
        rgba(0, 0, 0, 0.25) 50%
      ),
      linear-gradient(
        90deg,
        rgba(255, 0, 0, 0.06),
        rgba(0, 255, 0, 0.02),
        rgba(0, 0, 255, 0.06)
      );
    z-index: 2;
    background-size: 100% 4px, 3px 100%;
    animation: flicker 0.15s infinite, moveAnim 0.02s infinite linear;
    pointer-events: none;
  }

  @keyframes moveAnim {
    100% {
      top: 4px;
    }
  }

  @keyframes blur {
    0% {
      filter: blur(0.007em);
    }
    25% {
      filter: blur(0.021em);
    }
    100% {
      filter: blur(0.057em);
    }
  }

  @keyframes bg {
    28% {
      background: #112222;
    }
    30% {
      background: rgba(#112222, 0.95);
    }
    33% {
      background: #112222;
    }
    34% {
      background: rgba(#112222, 0.9);
    }
    35% {
      background: #112222;
    }
  }

  @keyframes flicker {
    0% {
      opacity: 0.27861;
    }
    5% {
      opacity: 0.34769;
    }
    10% {
      opacity: 0.23604;
    }
    15% {
      opacity: 0.90626;
    }
    20% {
      opacity: 0.18128;
    }
    25% {
      opacity: 0.83891;
    }
    30% {
      opacity: 0.65583;
    }
    35% {
      opacity: 0.67807;
    }
    40% {
      opacity: 0.26559;
    }
    45% {
      opacity: 0.84693;
    }
    50% {
      opacity: 0.96019;
    }
    55% {
      opacity: 0.08594;
    }
    60% {
      opacity: 0.20313;
    }
    65% {
      opacity: 0.71988;
    }
    70% {
      opacity: 0.53455;
    }
    75% {
      opacity: 0.37288;
    }
    80% {
      opacity: 0.71428;
    }
    85% {
      opacity: 0.70419;
    }
    90% {
      opacity: 0.7003;
    }
    95% {
      opacity: 0.36108;
    }
    100% {
      opacity: 0.24387;
    }
  }
</style>
