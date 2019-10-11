<script>
  import Bar from "./Bar.svelte";
  import GameMenu from "./GameMenu.svelte";
  import Ball from "./Ball.svelte";

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

  let ballPosition = { x: 50, y: 50 };
  let ballSpeedMultiplier;

  let ballXDirection;
  let ballXMomentum;


  $: ballXDirectionWithMomentum = ballXDirection + ballXMomentum;
  let ballYDirection = 1;

  function callbackFunction(event) {
    gameType = event.detail;
    initializeControls();
    createBall();
  }

  let resetGame = () => {
    gameType = undefined;
    ballSpeedMultiplier = 3;
    own = 0;
    enemy = 0;
    ballXDirection = 1;
    ballXMomentum = 0;
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
      clickTimeout = setTimeout(clickTimer, 10);
    };
    clickTimeout = setTimeout(clickTimer, 10);
  };

  let ballOverlapOwn = (b, r) => {
    return b.right >= r.left && b.left <= r.right && b.bottom >= r.top;
  };

  let ballOverlapEnemy = (b, r) => {
    return b.right >= r.left && b.left <= r.right && b.top <= r.bottom;
  };

  let ballTimeout;
  let clickTimeout;

  let calculateMomentum = (keys, ballXDirection, ballXMomentum) => {
    if (keys.rightDown) {
      if (ballXDirection === 1) {
        return ballXMomentum * -1 + 0.5;
      } else return ballXMomentum - 0.5;
    } else if (keys.leftDown) {
      if (ballXDirection === -1) {
        return ballXMomentum * -1 - 0.5;
      } else return ballXMomentum + 0.5;
    }

    return ballXMomentum;
  };

  let createBall = () => {
    let ballTimer = () => {
      if (gameType === undefined) return;

      ballPosition.x =
        ballPosition.x + ballXDirectionWithMomentum * ballSpeedMultiplier;
      ballPosition.y = ballPosition.y + ballYDirection * ballSpeedMultiplier;
      let ballElement = document.getElementById("ball").getBoundingClientRect();
      if (
        ballYDirection === 1 &&
        ballOverlapOwn(
          ballElement,
          document.getElementById("own").getBoundingClientRect()
        )
      ) {
        ballXMomentum = calculateMomentum(
          ownKeys,
          ballXDirection,
          ballXMomentum
        );

        ballYDirection = -ballYDirection;
        if (ballSpeedMultiplier < 10)
          ballSpeedMultiplier = ballSpeedMultiplier * 1.1;
      }
      if (
        ballYDirection === -1 &&
        ballOverlapEnemy(
          ballElement,
          document.getElementById("enemy").getBoundingClientRect()
        )
      ) {
        ballXMomentum = calculateMomentum(
          enemyKeys,
          ballXDirection,
          ballXMomentum
        );

        ballYDirection = -ballYDirection;
        if (ballSpeedMultiplier < 7)
          ballSpeedMultiplier = ballSpeedMultiplier * 1.1;
      } else {
        if (ballElement.left <= 0) {
          console.log("HIT LEFT WALL");
          ballXDirection = 1;
          ballXMomentum = ballXMomentum * -1;
        } else if (ballElement.right >= window.innerWidth) {
          console.log("HIT RIGHT WALL");
          ballXDirection = -1;
          ballXMomentum = ballXMomentum * -1;
        }

        if (ballElement.top <= 0 || ballElement.bottom >= window.innerHeight) {
          ballYDirection = -ballYDirection;
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
<Ball ball={ballPosition} />

<div>{ballXDirectionWithMomentum}</div>
<div>{ballXDirectionWithMomentum} = {ballXDirection} + {ballXMomentum}</div>
