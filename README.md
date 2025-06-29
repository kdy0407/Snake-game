# Snake-game
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<title>지렁이 게임 - 모바일 조이스틱</title>
<style>
  body {
    background: #222;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    height: 100vh;
    margin: 0;
    color: white;
    font-family: 'Arial', sans-serif;
    user-select: none;
    overflow: hidden;
    position: relative;
  }
  canvas {
    background: #111;
    border: 2px solid #0f0;
    border-radius: 10px;
    width: 90vw;
    height: 90vw;
    max-width: 400px;
    max-height: 400px;
    margin-top: 10px;
  }
  #score {
    font-size: 24px;
    margin: 10px;
  }
  #instructions {
    margin-top: 5px;
    font-size: 14px;
    color: #aaa;
    text-align: center;
    max-width: 90vw;
  }

  /* 조이스틱 스타일 */
  #joystick {
    position: fixed;
    bottom: 20px;
    left: 20px;
    width: 120px;
    height: 120px;
    background: rgba(255,255,255,0.1);
    border-radius: 50%;
    touch-action: none;
    user-select: none;
    z-index: 10;
  }
  #stick {
    position: absolute;
    left: 50%;
    top: 50%;
    width: 60px;
    height: 60px;
    margin-left: -30px;
    margin-top: -30px;
    background: rgba(0, 255, 0, 0.7);
    border-radius: 50%;
    transition: 0.1s ease;
    touch-action: none;
  }
</style>
</head>
<body>

<h1>지렁이 게임</h1>
<div id="score">점수: 0</div>
<canvas id="game" width="400" height="400"></canvas>
<div id="instructions">조이스틱으로 지렁이를 조작하세요</div>

<div id="joystick">
  <div id="stick"></div>
</div>

<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const gridSize = 20; 
  const tileCount = canvas.width / gridSize;

  let snake = [{x: 10, y: 10}];
  let velocity = {x: 0, y: 0};
  let food = {x: 15, y: 15};
  let score = 0;
  let gameOver = false;

  const joystick = document.getElementById('joystick');
  const stick = document.getElementById('stick');
  const maxDistance = 50; // 조이스틱 반지름 절반

  let dragging = false;
  let startX = 0;
  let startY = 0;

  function resetGame() {
    snake = [{x: 10, y: 10}];
    velocity = {x: 0, y: 0};
    placeFood();
    score = 0;
    gameOver = false;
    updateScore();
  }

  function placeFood() {
    food.x = Math.floor(Math.random() * tileCount);
    food.y = Math.floor(Math.random() * tileCount);
    while (snake.some(segment => segment.x === food.x && segment.y === food.y)) {
      food.x = Math.floor(Math.random() * tileCount);
      food.y = Math.floor(Math.random() * tileCount);
    }
  }

  function updateScore() {
    document.getElementById('score').textContent = `점수: ${score}`;
  }

  function gameLoop() {
    if (gameOver) {
      ctx.fillStyle = "rgba(0,0,0,0.7)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#f00";
      ctx.font = "40px Arial";
      ctx.textAlign = "center";
      ctx.fillText("게임 오버!", canvas.width / 2, canvas.height / 2 - 20);
      ctx.font = "20px Arial";
      ctx.fillText("스페이스바를 눌러 다시 시작", canvas.width / 2, canvas.height / 2 + 20);
      return;
    }

    let head = {x: snake[0].x + velocity.x, y: snake[0].y + velocity.y};

    if (head.x < 0) head.x = tileCount - 1;
    if (head.x >= tileCount) head.x = 0;
    if (head.y < 0) head.y = tileCount - 1;
    if (head.y >= tileCount) head.y = 0;

    if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
      gameOver = true;
    }

    if (velocity.x !== 0 || velocity.y !== 0) {
      snake.unshift(head);
      if (head.x === food.x && head.y === food.y) {
        score++;
        updateScore();
        placeFood();
      } else {
        snake.pop();
      }
    }

    ctx.fillStyle = "#111";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "#e00";
    ctx.fillRect(food.x * gridSize, food.y * gridSize, gridSize, gridSize);

    ctx.fillStyle = "#0f0";
    snake.forEach((segment, i) => {
      ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
      if(i === 0){
        ctx.fillStyle = "#7f7";
        ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
        ctx.fillStyle = "#0f0";
      }
    });
  }

  function keyDown(e) {
    if(gameOver && e.code === "Space"){
      resetGame();
      return;
    }
  }

  // 조이스틱 터치 시작
  joystick.addEventListener('touchstart', (e) => {
    e.preventDefault();
    dragging = true;
    const touch = e.touches[0];
    startX = touch.clientX;
    startY = touch.clientY;
    stick.style.transition = 'none';
  });

  // 조이스틱 터치 이동
  joystick.addEventListener('touchmove', (e) => {
    if (!dragging) return;
    e.preventDefault();
    const touch = e.touches[0];
    const dx = touch.clientX - startX;
    const dy = touch.clientY - startY;
    const dist = Math.sqrt(dx*dx + dy*dy);
    const angle = Math.atan2(dy, dx);

    const limitedDist = Math.min(dist, maxDistance);
    const stickX = limitedDist * Math.cos(angle);
    const stickY = limitedDist * Math.sin(angle);

    stick.style.transform = `translate(${stickX}px, ${stickY}px)`;

    // 방향 결정 (상하좌우 중 가장 가까운 방향)
    const absX = Math.abs(dx);
    const absY = Math.abs(dy);
    if (absX > absY) {
      if (dx > 0 && velocity.x !== -1) velocity = {x:1, y:0};
      else if (dx < 0 && velocity.x !== 1) velocity = {x:-1, y:0};
    } else {
      if (dy > 0 && velocity.y !== -1) velocity = {x:0, y:1};
      else if (dy < 0 && velocity.y !== 1) velocity = {x:0, y:-1};
    }
  });

  // 조이스틱 터치 끝
  joystick.addEventListener('touchend', (e) => {
    e.preventDefault();
    dragging = false;
    stick.style.transition = 'transform 0.2s ease';
    stick.style.transform = `translate(0,0)`;
  });

  window.addEventListener('keydown', keyDown);

  resetGame();
  setInterval(gameLoop, 200);
})();
</script>

</body>
</html>
