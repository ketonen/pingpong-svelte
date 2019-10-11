<script>
  import Bar from "./Bar.svelte";
  import GameMenu from "./GameMenu.svelte";
  import Ball from "./Ball.svelte";

  let gameType = "local";
  let own = 0;
  let enemy = 0;
  let ArrowRightDown = false;
  let ArrowLeftDown = false;
  let SKeyDown = false;
  let AKeyDown = false;

  let ballPosition = { x: 50, y: 50 };
  let ballXDirection = 1;
  let ballYDirection = 1;

  function callbackFunction(event) {
    gameType = event.detail;
    console.log(gameType);
  }

  let initializeControls = () => {
    document.onkeydown = e => {
      if (e.key === "ArrowRight") ArrowRightDown = true;
      if (e.key === "ArrowLeft") ArrowLeftDown = true;
      if (gameType === "local") {
        if (e.key === "s") SKeyDown = true;
        if (e.key === "a") AKeyDown = true;
      }
    };
    document.onkeyup = e => {
      if (e.key === "ArrowRight") ArrowRightDown = false;
      if (e.key === "ArrowLeft") ArrowLeftDown = false;
      if (gameType === "local") {
        if (e.key === "s") SKeyDown = false;
        if (e.key === "a") AKeyDown = false;
      }
    };

    let clickTimer = () => {
      if (ArrowRightDown) own++;
      if (ArrowLeftDown) own--;
      if (SKeyDown) enemy++;
      if (AKeyDown) enemy--;
      clickTimeout = setTimeout(clickTimer, 10);
    };
    clickTimeout = setTimeout(clickTimer, 10);
  };

  let ballOverlapOwn = (ball, rect2) => {
    let b = ball.getBoundingClientRect();
    let r = rect2.getBoundingClientRect();

    return b.right >= r.left && b.left <= r.right && b.bottom >= r.top;
  };

  let ballOverlapEnemy = (ball, rect2) => {
    let b = ball.getBoundingClientRect();
    let r = rect2.getBoundingClientRect();
    return b.right >= r.left && b.left <= r.right && b.top <= r.bottom;
  };

  let ballTimeout;
  let clickTimeout;
  let ballSpeedMultiplier = 3;

  let createBall = () => {
    let ballTimer = () => {
      ballPosition.x = ballPosition.x + ballXDirection * ballSpeedMultiplier;
      ballPosition.y = ballPosition.y + ballYDirection * ballSpeedMultiplier;
      let ballElement = document.getElementById("ball");
      if (
        (ballYDirection === 1 &&
          ballOverlapOwn(ballElement, document.getElementById("own"))) ||
        (ballYDirection === -1 &&
          ballOverlapEnemy(ballElement, document.getElementById("enemy")))
      ) {
        ballYDirection = -ballYDirection;
        if (ballSpeedMultiplier < 10)
          ballSpeedMultiplier = ballSpeedMultiplier * 1.1;
      } else {
        if (ballPosition.x - 5 < 0 || ballPosition.x + 20 > window.innerWidth) {
          ballXDirection = -ballXDirection;
        }
        if (
          ballPosition.y - 5 < 0 ||
          ballPosition.y + 20 > window.innerHeight
        ) {
          ballYDirection = -ballYDirection;
          gameType = undefined;
          clearTimeout(clickTimeout);
          clearTimeout(ballTimeout);
        }
      }

      ballTimeout = setTimeout(ballTimer, 10);
    };
    ballTimeout = setTimeout(ballTimer, 10);
  };

  initializeControls();
  createBall();
</script>

{#if gameType === undefined}
  <GameMenu on:gameModeSelected={callbackFunction} />
{:else}
  <Bar type={'own'} position={own} />
  <Bar type={'enemy'} position={enemy} />
  <Ball ball={ballPosition} />
{/if}
