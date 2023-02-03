{% include arcadeSubmenu.html %}

<!DOCTYPE html>
<html>
<head>
  <style>
    #game-board {
      width: 500px;
      height: 500px;
      border: 1px solid black;
    }
  </style>
</head>
<body>
  <button id="start-btn">Start Game</button>
  <div id="game-board"></div>
  <div id="game-over" style="display: none;">
    <p>Game Over</p>
    <button id="retry-btn">Retry</button>
  </div>
</body>
<script src="snake.js"></script>
</html>



<style>
    #game-board {
    width: 500px;
    height: 500px;
    border: 1px solid black;
  }

  .snake-unit {
    width: 10px;
    height: 10px;
    position: absolute;
    background-color: green;
  }

  .food-unit {
    width: 10px;
    height: 10px;
    position: absolute;
    background-color: red;
  }
</style>

<script>
  const startBtn = document.getElementById("start-btn");
  const retryBtn = document.getElementById("retry-btn");
  const gameBoard = document.getElementById("game-board");
  const gameOver = document.getElementById("game-over");

  let snake = [];
  let food = null;
  let direction = "right";
  let gameInterval = null;

  const createSnake = () => {
    snake = [];
    for (let i = 4; i >= 0; i--) {
      snake.push({ x: i, y: 0 });
    }
  };

  const createFood = () => {
    food = {
      x: Math.floor(Math.random() * (gameBoard.offsetWidth / 10)),
      y: Math.floor(Math.random() * (gameBoard.offsetHeight / 10)),
    };
  };

  const drawSnake = () => {
    snake.forEach((unit) => {
      let snakeUnit = document.createElement("div");
      snakeUnit.classList.add("snake-unit");
      snakeUnit.style.left = unit.x * 10 + "px";
      snakeUnit.style.top = unit.y * 10 + "px";
      gameBoard.appendChild(snakeUnit);
    });
  };

  const drawFood = () => {
    let foodUnit = document.createElement("div");
    foodUnit.classList.add("food-unit");
    foodUnit.style.left = food.x * 10 + "px";
    foodUnit.style.top = food.y * 10 + "px";
    gameBoard.appendChild(foodUnit);
  };

  const moveSnake = () => {
    let nextX = snake[0].x;
    let nextY = snake[0].y;

    switch (direction) {
      case "right":
        nextX++;
        break;
      case "left":
        nextX--;
        break;
      case "down":
        nextY--;
        break;
      case "up":
        nextY++;
        break;
      }

    if (
      nextX < 0 ||
      nextX >= gameBoard.offsetWidth / 10 ||
      nextY < 0 ||
      nextY >= gameBoard.offsetHeight / 10 ||
      snake.some((unit) => unit.x === nextX && unit.y === nextY)
    ) {
      clearInterval(gameInterval);
      gameOver.style.display = "block";
      return;
    }

    if (nextX === food.x && nextY === food.y) {
      food = null;
      createFood();
      drawFood();
    } else {
      snake.pop();
    }

    snake.unshift({ x: nextX, y: nextY });
    gameBoard.innerHTML = "";
    drawFood();
    drawSnake();
  };

  startBtn.addEventListener("click", () => {
    createSnake();
    createFood();
    drawFood();
    drawSnake();
    gameInterval = setInterval(moveSnake, 100);
  });

  retryBtn.addEventListener("click", () => {
    gameOver.style.display = "none";
    createSnake();
    createFood();
    drawFood();
    drawSnake();
    gameInterval = setInterval(moveSnake, 100);
  });

  document.addEventListener("keydown", (e) => {
    switch (e.code) {
      case "ArrowLeft":
        if (direction === "left") return;
        direction = "left";
        break;
      case "ArrowUp":
        if (direction === "up") return;
        direction = "up";
        break;
      case "ArrowRight":
        if (direction === "right") return;
        direction = "right";
        break;
      case "ArrowDown":
        if (direction === "down") return;
        direction = "down";
        break;
    }
  });
</script>


