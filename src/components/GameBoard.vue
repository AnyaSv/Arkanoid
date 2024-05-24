<template>
  <div class="game-board">
    <div class="board-row" v-for="(row, rowIdx) in Board" :key="rowIdx">
      <div
        class="board-cell"
        :class="{
          'brick': cell == 1,
          'ball': cell == 2,
          'rocket': cell == 3
        }"
        v-for="(cell, cellIdx) in row"
        :key="cellIdx"
      >
        &nbsp;
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const Board = ref([]);
const Rocket = ref({
  x: 5,
  y: 24,
  width: 3,
  height: 1
});

const ball = ref({
  x: 6,
  y: 12,
  radius: 1,
  dx: 1,
  dy: -1
});

const bricks = ref([]);
let interval;

onMounted(() => {
  InitBoardAndBricks();
  window.addEventListener('keydown', onKeyDown);
  interval = setInterval(() => moveBall(), 100);
});


function onKeyDown(evt) {
  switch (evt.key) {
    case 'ArrowLeft':
      MoveRocket(-1);
      break;
    case 'ArrowRight':
      MoveRocket(1);
      break;
  }
}

function MoveRocket(drctn) {
  const newX = Rocket.value.x + drctn;

  if (newX >= 0 && newX + Rocket.value.width <= 13) {
    eraseRocket();
    Rocket.value.x = newX;
    drawRocket();
  }
}

function eraseRocket() {
  for (let i = 0; i < Rocket.value.width; i++) {
    const cellX = Rocket.value.x + i;
    if (cellX >= 0 && cellX < 13) {
      Board.value[Rocket.value.y][cellX] = 0;
    }
  }
}

function drawRocket() {
  for (let i = 0; i < Rocket.value.width; i++) {
    const cellX = Rocket.value.x + i;
    if (cellX >= 0 && cellX < 13) {
      Board.value[Rocket.value.y][cellX] = 3;
    }
  }
}

function InitBoardAndBricks() {
  InitBoard();
  InitBricks();
}

function InitBoard() {
  Board.value = [];

  for (let y = 0; y < 25; y++) {
    let row = [];
    for (let x = 0; x < 13; x++) {
      row.push(0);
    }
    Board.value.push(row);
  }

  drawRocket();
}

function InitBricks() {
  bricks.value = [
    [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1],
    [0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0],
    [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1]
  ];
  drawRocket();
}

function drawBricks() {
  for (let y = 0; y < bricks.value.length; y++) {
    for (let x = 0; x < bricks.value[y].length; x++) {
      if (bricks.value[y][x] === 1) {
        Board.value[y][x] = 1;
      }
    }
  }
}

function handleBrickCollision(brickY, brickX) {
  bricks.value[brickY][brickX] = 0;

  if (ball.value.dx > 0 && ball.value.dy > 0) {
    ball.value.dx = -ball.value.dx;
  } else if (ball.value.dx < 0 && ball.value.dy > 0) {
    ball.value.dy = -ball.value.dy;
  } else if (ball.value.dx > 0 && ball.value.dy < 0) {
    ball.value.dy = -ball.value.dy;
  } else if (ball.value.dx < 0 && ball.value.dy < 0) {
    ball.value.dx = -ball.value.dx;
  }
}

function eraseBall() {
  const ballX = Math.floor(ball.value.x);
  const ballY = Math.floor(ball.value.y);
  if (ballX >= 0 && ballX < 13 && ballY >= 0 && ballY < 25) {
    Board.value[ballY][ballX] = 0;
  }
}

function drawBall() {
  const ballX = Math.floor(ball.value.x);
  const ballY = Math.floor(ball.value.y);
  if (ballX >= 0 && ballX < 13 && ballY >= 0 && ballY < 25) {
    Board.value[ballY][ballX] = 2;
  }
}

function moveBall() {
  eraseBall();

  const nextX = ball.value.x + ball.value.dx;
  const nextY = ball.value.y + ball.value.dy;

  if (nextX < 0 || nextX > 12) {
    ball.value.dx = -ball.value.dx;
  }

  if (nextY < 0) {
    ball.value.dy = -ball.value.dy;
  }

  if (nextY > 24) {
    GameOver();
    return;
  }

  if (noMoreBricks()) {
    GameOver();
    return;
  }

  if (
    nextY === Rocket.value.y &&
    nextX >= Rocket.value.x &&
    nextX <= Rocket.value.x + Rocket.value.width
  ) {
    ball.value.dy = -ball.value.dy;
  }

  const brickX = Math.floor(nextX);
  const brickY = Math.floor(nextY);

  if (
    brickY >= 0 &&
    brickY < bricks.value.length &&
    brickX >= 0 &&
    brickX < bricks.value[brickY].length &&
    bricks.value[brickY][brickX] === 1
  ) {
    handleBrickCollision(brickY, brickX);
  }

  ball.value.x = nextX;
  ball.value.y = nextY;

  drawBall();
  drawRocket();
  drawBricks();
}

function noMoreBricks() {
  for (let y = 0; y < bricks.value.length; y++) {
    for (let x = 0; x < bricks.value[y].length; x++) {
      if (bricks.value[y][x] === 1) {
        return false;
      }
    }
  }
  return true;
}

function GameOver() {
  clearInterval(interval);
  console.log("Game Over");
  alert("Game Over");
}
</script>

<style>
.game-board {
  margin: 50px;
}

.board-cell {
  width: 20px;
  height: 20px;
  border: 1px solid #ccc;
  display: inline-block;
}

.brick {
  background-color: brown;
}

.ball {
  background-color: red;
  border-radius: 50%;
}

.rocket {
  background-color: blue;
}
</style>