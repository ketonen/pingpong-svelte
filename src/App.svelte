<script>
  import Bar from "./Bar.svelte";
  import GameMenu from "./GameMenu.svelte";
  import Ball from "./Ball.svelte";
  import { spring } from "svelte/motion";

  let gameType;
  let own;
  let enemy;
  let ownKeys = {
    rightDown: false,
    leftDown: false
  };
  let enemyKeys = {
    leftDown: false,
    rightDown: false
  };

  let ballPosition;
  let ballCoords;
  $: ballCoords.set({ x: ballPosition.x + ballStep.x, y: ballPosition.y + ballStep.y })

  let ballStep = {
    x: 0,
    y: 2
  };

  function callbackFunction(event) {
    gameType = event.detail;
    initializeControls();
    createBall();
  }

  let resetGame = () => {
    ballPosition = { x: 50, y: 50 };

    ballCoords = spring(
      { x: ballPosition.x, y: ballPosition.y },
      {
        stiffness: 0.5,
        damping: 1
      }
    );
    gameType = undefined;
    own = 0;
    enemy = 0;
  };

  let initializeControls = () => {
    document.onkeydown = e => {
      if (e.key === "ArrowRight") ownKeys.rightDown = true;
      if (e.key === "ArrowLeft") ownKeys.leftDown = true;
      if (gameType === "local") {
        if (e.key === "s") enemyKeys.rightDown = true;
        if (e.key === "a") enemyKeys.leftDown = true;
      }
    };
    document.onkeyup = e => {
      if (e.key === "ArrowRight") ownKeys.rightDown = false;
      if (e.key === "ArrowLeft") ownKeys.leftDown = false;
      if (gameType === "local") {
        if (e.key === "s") enemyKeys.rightDown = false;
        if (e.key === "a") enemyKeys.leftDown = false;
      }
    };

    let clickTimer = () => {
      if (ownKeys.rightDown) own++;
      if (ownKeys.leftDown) own--;
      if (enemyKeys.rightDown) enemy++;
      if (enemyKeys.leftDown) enemy--;
      clickTimeout = setTimeout(clickTimer, 20);
    };
    clickTimeout = setTimeout(clickTimer, 20);
  };

  let ballOverlapOwn = (b, r) => {
    return b.right >= r.left && b.left <= r.right && b.bottom >= r.top;
  };

  let ballOverlapEnemy = (b, r) => {
    return b.right >= r.left && b.left <= r.right && b.top <= r.bottom;
  };

  let increaseBallSpeed = ballStep => {
    let increateSpeed = axisStep => {
      if (Math.abs(axisStep) >= 7) return axisStep;
      if (axisStep < 0) return axisStep - 1;
      if (axisStep > 0) return axisStep + 1;
      return axisStep;
    };

    ballStep.x = increateSpeed(ballStep.x);
    ballStep.y = increateSpeed(ballStep.y);

    return ballStep;
  };

  let ballTimeout;
  let clickTimeout;

  let increateMomentum = keys => {
    if (keys.rightDown) ballStep.x = ballStep.x + 1;
    else if (keys.leftDown) ballStep.x = ballStep.x - 1;
  };

  let createBall = () => {
    let ballTimer = () => {
      if (gameType === undefined) return;

      ballPosition.x = ballPosition.x + ballStep.x;
      ballPosition.y = ballPosition.y + ballStep.y;
      let ballElement = document.getElementById("ball").getBoundingClientRect();
      if (
        ballStep.y > 0 &&
        ballOverlapOwn(
          ballElement,
          document.getElementById("own").getBoundingClientRect()
        )
      ) {
        ballStep.y = -ballStep.y;
        ballStep = increaseBallSpeed(ballStep);
        increateMomentum(ownKeys);
      }
      if (
        ballStep.y < 0 &&
        ballOverlapEnemy(
          ballElement,
          document.getElementById("enemy").getBoundingClientRect()
        )
      ) {
        ballStep.y = -ballStep.y;
        ballStep = increaseBallSpeed(ballStep);
        increateMomentum(enemyKeys);
      } else {
        if (
          (ballStep.x < 0 && ballElement.left <= 0) ||
          (ballStep.x > 0 && ballElement.right >= window.innerWidth)
        ) {
          console.log("HIT WALL");
          ballStep.x = -ballStep.x;
        }

        if (ballElement.top <= 0 || ballElement.bottom >= window.innerHeight) {
          resetGame();
          clearTimeout(clickTimeout);
          clearTimeout(ballTimeout);
        }
      }

      ballTimeout = setTimeout(ballTimer, 10);
    };
    ballTimeout = setTimeout(ballTimer, 10);
  };

  resetGame();
</script>

{#if gameType === undefined}
  <GameMenu on:gameModeSelected={callbackFunction} />
{:else}
  <Bar type={'own'} position={own} />
  <Bar type={'enemy'} position={enemy} />
{/if}
<Ball coords={ballCoords} />
