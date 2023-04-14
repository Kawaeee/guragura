<script lang="ts">
    // Player parameters
    let startPosition: number = 0;
    let movePowerPixel: number = 50;

    // Game parameters
    let isGameStarted: boolean = false;
    let lastGameStatus: boolean = false;

    let startScore: number = 0;
    let scoreMultiplier = 1.0;

    let remainingTime: number = 60;
    let timerInterval: any = null;

    let goalReachedCount: number = 0;

    // Goal parameters
    const goalReachedAdditionalTime: number = 10;
    let goalReachedBaseScore: number = 10;
    const goalReachedUpgradeTrigger: number = 3;
    const goalOverlapThreshold: number = 0.2;
    
    // Key bindings
    let lastKeyPressed: string = "";

    function getIntersectionRatio(rect1 : any, rect2: any): number {
        const xOverlap = Math.max(0, Math.min(rect1.right, rect2.right) - Math.max(rect1.left, rect2.left));
        const yOverlap = Math.max(0, Math.min(rect1.bottom, rect2.bottom) - Math.max(rect1.top, rect2.top));
        const intersectionArea = xOverlap * yOverlap;
        const rect1Area = (rect1.right - rect1.left) * (rect1.bottom - rect1.top);
        return intersectionArea / rect1Area;
    }

    function handleKeyDown(event: { key: any; }) {
        const currentKey = event.key;

        if (currentKey === "ArrowLeft" && lastKeyPressed !== "ArrowLeft") {
            startPosition += movePowerPixel;
            lastKeyPressed = "ArrowLeft";
        } else if (currentKey === "ArrowRight" && lastKeyPressed !== "ArrowRight") {
            startPosition += movePowerPixel;
            lastKeyPressed = "ArrowRight";
        }

        const player = document.querySelector('.player');
        const goal = document.querySelector('.goal');

        // Get player & goal bbox
        const playerRect = player.getBoundingClientRect();
        const goalRect = goal.getBoundingClientRect();
        
        const intersection = getIntersectionRatio(playerRect, goalRect);

        if (intersection > goalOverlapThreshold) {
            goalReachedCount++;
            startScore += goalReachedBaseScore * scoreMultiplier;
            startPosition = 0;
            lastKeyPressed = "";

            if (goalReachedCount % goalReachedUpgradeTrigger === 0) {
                movePowerPixel -= 1;
                
                // In case of it reach hard limit
                if(movePowerPixel < 5){
                    movePowerPixel = 30;
                }

            // Random upgrade base score and multiplier
            goalReachedBaseScore += Math.floor(Math.random() * 10);
            scoreMultiplier += Math.random();
            remainingTime += goalReachedAdditionalTime;
            }
      }
    }

    // Timer state functions
    function startTimer() {
        timerInterval = setInterval(() => {
        if (remainingTime > 0) {
            remainingTime--;
        } else {
            stopGame();
        }
        }, 1000);
    }

    function stopTimer() {
        clearInterval(timerInterval);
        timerInterval = null;
    }

    // Game state functions
    function startGame() {
        isGameStarted = true;
        lastGameStatus = false;
        startTimer();
        // Trigger restart to remove existing values
        restartGame();
        window.addEventListener('keydown', handleKeyDown);
    }

    function restartGame(){
        startPosition = 0;
        movePowerPixel = 50;

        startScore = 0;
        goalReachedBaseScore = 10;
        scoreMultiplier = 1.0;

        remainingTime = 10;
        goalReachedCount = 0;
    }

    function stopGame() {
        isGameStarted = false;
        lastGameStatus = true;
        stopTimer();
        window.removeEventListener('keydown', handleKeyDown);
    }
</script>

{#if !isGameStarted}
  <button on:click={startGame}>Start Game</button>
  {#if lastGameStatus}
    Lap: {goalReachedCount}, Score: {startScore.toFixed(2)}, Multiplier: x{scoreMultiplier.toFixed(2)}
  {/if}
{/if}

{#if isGameStarted}
<div class="guragura-game">
  <div>Lap: {goalReachedCount}, Score: {startScore.toFixed(2)}, Multiplier: x{scoreMultiplier.toFixed(2)}</div>
  <div>Time remaining: {remainingTime} seconds</div>
  <div class="player" style="left: {startPosition}px;"></div>
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
  
    .guragura-game {
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