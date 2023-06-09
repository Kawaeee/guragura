<!-- 
* This Guragura.svelte provides a simple game where the player using the left and right arrow keys. 
* The goal of the game is to reach the goal (a static element on the screen) as many times as 
* possible before the time runs out. Each time the goal is reached, the player gains points and
* the game difficulty increases. The player can overlap with the goal if a certain overlap threshold is met.
-->

<script lang="ts">
    import { fade } from "svelte/transition";
    import { onMount } from "svelte";

    import playerImage from "$lib/images/player.gif";
    import goalImage from "$lib/images/goal.png";

    /**
     * The starting position of the player in pixels.
     * @type {number}
     */
    let startPosition = 0;

    /**
     * The amount of pixels the player moves left or right with each key press.
     * @type {number}
     */
    let movePowerPixel = 50;

    /**
     * A boolean value indicating whether the game has started or not.
     * @type {boolean}
     */
    let isGameStarted = false;

    /**
     * A boolean value indicating the previous game state. 
     * @type {boolean}
     */
    let lastGameStatus = false;

    /**
     * The player's score at the start of the game.
     * @type {number}
     */
    let startScore = 0;

    /**
     * A multiplier applied to the score when the goal is reached.
     * @type {number}
     */
    let scoreMultiplier = 1.0;

    /**
     * The amount of time left in the game in seconds.
     * @type {number}
     */
    let remainingTime = 60;

    /**
     * The amount of time waste on the game in seconds xD.
     * @type {number}
     */
    let cumulativeTimeSpent = 0;

    /**
     * The ID of the timer interval function.
     * @type {number}
     */
    let timerInterval = null;

    /**
     * The number of times the player has reached the goal.
     * @type {number}
     */
    let goalReachedCount = 0;

    /**
     * The amount of additional time in seconds awarded to the player when the goal is reached.
     * @type {number}
     */
    const goalReachedAdditionalTime = 10;

    /**
     * The base score awarded to the player when the goal is reached.
     * @type {number}
     */
    let goalReachedBaseScore = 10;

    /**
     * The number of goal reached count triggers required to upgrade the game difficulty.
     * @type {number}
     */
    const goalReachedUpgradeTrigger = 3;

    /**
     * The intersection threshold required for the player to overlap with the goal.
     * @type {number}
     */
    const goalOverlapThreshold = 0.2;

    /**
     * The last key pressed by the player.
     * @type {string}
     */
    let lastKeyPressed = "";

    /**
     * Calculates the intersection ratio of two elements.
     * @param {Object} rect1 The bounding rectangle of the first element.
     * @param {Object} rect2 The bounding rectangle of the second element.
     * @returns {number} The intersection ratio of the two elements.
     */
    function getIntersectionRatio(rect1: any, rect2: any) {
        const xOverlap = Math.max(0, Math.min(rect1.right, rect2.right) - Math.max(rect1.left, rect2.left));
        const yOverlap = Math.max(0, Math.min(rect1.bottom, rect2.bottom) - Math.max(rect1.top, rect2.top));
        const intersectionArea = xOverlap * yOverlap;
        const rect1Area = (rect1.right - rect1.left) * (rect1.bottom - rect1.top);
        return intersectionArea / rect1Area;
    }

    /**
     * Returns the HTML element for the player.
     * @return {HTMLElement} The HTML element for the player.
     */
    function getPlayer() {
        return document.querySelector(".player");
    }

    /**
     * Returns the HTML element for the goal.
     * @return {HTMLElement} The HTML element for the goal.
     */
    function getGoal() {
        return document.querySelector(".goal");
    }

    /**
     * Checks if the player element and goal element overlap.
     * @param {HTMLElement} player - The HTML element for the player.
     * @param {HTMLElement} goal - The HTML element for the goal.
     * @return {boolean} Whether the player and goal elements overlap.
     */
    function isPlayerOverlappingGoal(player: HTMLElement, goal: HTMLElement) {
        const playerRect = player.getBoundingClientRect();
        const goalRect = goal.getBoundingClientRect();
        const intersection = getIntersectionRatio(playerRect, goalRect);
        return intersection > goalOverlapThreshold;
    }

    /**
     * Handles keydown events and updates the player position and checks for goal overlap.
     * @param {object} event - The keydown event.
     * @param {string} event.key - The key that was pressed.
     */
    function handleKeyDown(event: { key: any; }) {
        const currentKey = event.key;
        const player = getPlayer();
        const goal = getGoal();

        if (currentKey === "ArrowLeft" && lastKeyPressed !== "ArrowLeft") {
            startPosition += movePowerPixel;
            lastKeyPressed = "ArrowLeft";
        } else if (currentKey === "ArrowRight" && lastKeyPressed !== "ArrowRight") {
            startPosition += movePowerPixel;
            lastKeyPressed = "ArrowRight";
        }

        if (isPlayerOverlappingGoal(player, goal)) {
            handleGoalReached();
        }
    }

    /**
     * Handles when the player reaches the goal and upgrades the game difficulty and score multiplier.
     */
    function handleGoalReached() {
        goalReachedCount++;
        updateScoreAndPosition();
        resetLastKeyPressed();
        handleGoalUpgrade();
    }

    /**
     * Updates the player's score and position after the player reaches the goal.
     */
    function updateScoreAndPosition() {
        startScore += goalReachedBaseScore * scoreMultiplier;
        startPosition = 0;
    }

    /**
     * Resets the last key pressed by the player.
     */
    function resetLastKeyPressed() {
        lastKeyPressed = "";
    }

    /**
     * Handles upgrading the game difficulty when the player reaches a certain number of goals.
     */
    function handleGoalUpgrade() {
        if (goalReachedCount % goalReachedUpgradeTrigger === 0) {
            upgradeGameDifficulty();
            upgradeScoreAndMultiplier();
            addTimeToGame();
        }
    }

    /**
     * Upgrades the game difficulty by decreasing the move power pixel.
     */

    function upgradeGameDifficulty() {
        movePowerPixel = Math.max(movePowerPixel - 1, 5);
    }

    /**
     * Upgrades the goal score and multiplier by a random amount.
     */
    function upgradeScoreAndMultiplier() {
        goalReachedBaseScore += Math.floor(Math.random() * 10);
        scoreMultiplier += Math.random();
    }

    /**
     * Adds additional time to the game when the player reaches a certain number of goals.
     */
    function addTimeToGame() {
        remainingTime += goalReachedAdditionalTime;
        cumulativeTimeSpent += goalReachedAdditionalTime;
    }

    /**
     * Starts the game timer.
     */
    function startTimer() {
        timerInterval = setInterval(() => {
        if (remainingTime > 0) {
            remainingTime--;
        } else {
            stopGame();
        }
        }, 1000);
    }

    /**
     * Stop the game timer.
     */
    function stopTimer() {
        clearInterval(timerInterval);
        timerInterval = null;
    }

    /**
     * Starts the game by setting the game state and starting the timer and event listener.
     */
    function startGame() {
        isGameStarted = true;
        lastGameStatus = false;
        startTimer();
        // Trigger restart to remove existing values
        restartGame();
        window.addEventListener("keydown", handleKeyDown);
    }

    /**
     * Restarts the game by resetting game parameters and removing the event listener.
     */
    function restartGame(){
        startPosition = 0;
        movePowerPixel = 50;

        startScore = 0;
        goalReachedBaseScore = 10;
        scoreMultiplier = 1.0;

        remainingTime = 60;
        cumulativeTimeSpent = 60;
        goalReachedCount = 0;
    }

    /**
     * Stops the game by setting the game state, stopping the timer, and removing the event listener.
     */
    function stopGame() {
        isGameStarted = false;
        lastGameStatus = true;
        stopTimer();
        window.removeEventListener("keydown", handleKeyDown);
    }

    // This is placeholder for smaller screen
    let isSmallScreen = false;

    const updateScreenSize = () => {
        if (window.innerWidth <= 500) {
            isSmallScreen = true;
        } else {
            isSmallScreen = false;
        }
    };

    onMount(() => {
        updateScreenSize();
        window.addEventListener("resize", updateScreenSize);
    });

</script>

{#if isSmallScreen}
<h5 class="message">Try this on larger screen would be 🔥🥕;</h5>
{/if}

{#if !isGameStarted}
<div class="guragura-screen" transition:fade>
    <h1>Ready to Guragura? Start Moving Now!</h1>
    <button class="start-btn" on:click={startGame}>Run!</button>
    {#if lastGameStatus}
    <div class="last-score">
        Lap: {goalReachedCount}
        <br>Score: <span class="blue">{startScore.toFixed(2)}</span>
        <br>Multiplier: <span class="green">x{scoreMultiplier.toFixed(2)}</span>
        <br>You have wasted <span class="red">{cumulativeTimeSpent} seconds</span> for previous run.
    </div>
    {/if}
</div>
{/if}

{#if isGameStarted}
<div class="guragura-screen" transition:fade>
    <div class="current-score">
        Lap: {goalReachedCount}
        <br>Score: <span class="blue">{startScore.toFixed(2)}</span>
        <br>Multiplier: <span class="green">x{scoreMultiplier.toFixed(2)}</span>
        <br>Time remaining: <span class="red">{remainingTime} seconds</span>
    </div>  
    <div class="game-content">
        <div class="player" style="left: {startPosition}px;">
            <img src={playerImage} alt="player">
        </div>
        <div class="goal">
            <img src={goalImage} alt="goal" width=50 height=50>
        </div>
    </div>
    <p class="instruction">Press the left and right arrow keys alternately to move the player!</p>
</div>
{/if}

<style>
    .message {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
    }

    .guragura-screen {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 100%;
        max-width: 700px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
    }

    .red {
        color: #FFCCCB;
    }

    .green {
        color: lightgreen;
    }

    .blue {
        color: lightblue;
    }

    .last-score{
        position: absolute;
        bottom: -100px;
        font-size: medium;
    }

    .current-score{
        position: absolute;
        top: 100px;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: medium;
    }

    .game-content {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        white-space: nowrap;
    }

    .instruction {
        margin-top: 20px;
    }

    .start-btn {
        background-color: white;
        border: none;
        color: #1F242A;
        padding: 10px 20px;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        font-size: 16px;
        margin: 4px 2px;
        cursor: pointer;
        border-radius: 12px;
    }

    .player {
        position: absolute;
        bottom: 150%;
        left: 0%;
        width: 32px;
        height: 60px;
        transform: translate(-50%, -50%);
    }
  
    .goal {
        position: absolute;
        bottom: 150%;
        left: 100%;
        transform: translate(-50%, -50%);
        width: 50px;
        height: 50px;
    }

    @media (max-width: 500px) {
        .guragura-screen {
            display: none;
        }
    }
</style>