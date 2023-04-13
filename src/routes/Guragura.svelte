<script>
  let position = 0;
  let movePower = 50;

  let score = 0;
  let baseScore = 10;
  let scoreMultiplier = 1.0;

  let timeRemaining = 60;
  const goalReachedAddTime = 10;

  let lastKeyPressed = null;
  let timerIntervalId = null;
  let goalReachedCount = 0;

  let gameStarted = false;
  let lastGameStatus = false;

  function handleKeyDown(event) {
    if (event.key === 'ArrowLeft' && lastKeyPressed !== 'ArrowLeft') {
      position += movePower;
      lastKeyPressed = 'ArrowLeft';
    } else if (event.key === 'ArrowRight' && lastKeyPressed !== 'ArrowRight') {
      position += movePower;
      lastKeyPressed = 'ArrowRight';
    }

    const player = document.querySelector('.player');
    const goal = document.querySelector('.goal');

    const playerRect = player.getBoundingClientRect();
    const goalRect = goal.getBoundingClientRect();

    const intersection = getIntersectionRatio(playerRect, goalRect);

    if (intersection > 0.5) {
        goalReachedCount++;
        score += baseScore * scoreMultiplier;
        position = 0;
        lastKeyPressed = null;

        if (goalReachedCount % 3 === 0) {
          movePower -= 1;
          
          if(movePower < 0){
            movePower = 30;
          }

          baseScore += Math.floor(Math.random() * 10);
          scoreMultiplier += Math.random();
          timeRemaining += goalReachedAddTime;
        }
      }
  }

function getIntersectionRatio(rect1, rect2) {
  const xOverlap = Math.max(0, Math.min(rect1.right, rect2.right) - Math.max(rect1.left, rect2.left));
  const yOverlap = Math.max(0, Math.min(rect1.bottom, rect2.bottom) - Math.max(rect1.top, rect2.top));
  const intersectionArea = xOverlap * yOverlap;
  const rect1Area = (rect1.right - rect1.left) * (rect1.bottom - rect1.top);
  return intersectionArea / rect1Area;
}

function startTimer() {
    timerIntervalId = setInterval(() => {
      if (timeRemaining > 0) {
        timeRemaining--;
      } else {
        stopGame();
      }
    }, 1000);
  }


  function stopTimer() {
    clearInterval(timerIntervalId);
    timerIntervalId = null;
  }

  function startGame() {
    gameStarted = true;
    lastGameStatus = false;
    startTimer();
    restartGame();
    window.addEventListener('keydown', handleKeyDown);
  }

  function restartGame(){
    position = 0;
    movePower = 50;
    score = 0;
    baseScore = 10;
    scoreMultiplier = 1.0;
    timeRemaining = 10;
    goalReachedCount = 0;
  }

  function stopGame() {
    gameStarted = false;
    lastGameStatus = true;
    stopTimer();
    window.removeEventListener('keydown', handleKeyDown);
  }

</script>

{#if !gameStarted}
  <button on:click={startGame}>Start Game</button>
  {#if lastGameStatus}
    Lap: {goalReachedCount}, Score: {score.toFixed(2)}, Multiplier: x{scoreMultiplier.toFixed(2)}
  {/if}
{/if}

{#if gameStarted}
<div class="guragura-screen">
  <div>Lap: {goalReachedCount}, Score: {score.toFixed(2)}, Multiplier: x{scoreMultiplier.toFixed(2)}</div>
  <div>Time remaining: {timeRemaining} seconds</div>
  <div class="player" style="left: {position}px;"></div>
  <div class="goal"></div>
  <slot></slot>
  <p>Press the left and right arrow keys alternately to move the player!</p>
</div>
{/if}

<style>
  .player {
      position: absolute;
      bottom: 150%;
      left: 0%;
      transform: translate(-50%, -50%);
      width: 50px;
      height: 50px;
      background-color: blue;
  }

  .goal {
    position: absolute;
    bottom: 150%;
    left: 100%;
    transform: translate(-50%, -50%);
    width: 50px;
    height: 50px;
    background-color: red;
  }

  .guragura-screen {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 100%;
      max-width: 500px; /* or any other maximum width you want */
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      text-align: center;
      z-index: 0;
  }
</style>
