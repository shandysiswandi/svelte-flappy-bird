<script>
  import { onMount } from "svelte";

  // Score info
  const score = {
    sound: new Audio(),
    current: 0,
  };

  // Canvas and Context 2D
  const canvasWidth = 500;
  const canvasHeight = 700;
  let canvas;
  let context;
  let frame;

  // Bird Info
  const bird = new Image();
  const birdFlySound = new Audio();
  const birdSpeed = 7;
  const birdSize = 50;
  const birdDx = 0; // Bord's speed in x direction
  let birdDy = 0; // Bird's speed in y direction
  let birdX = 50; // Bird's X coord
  let birdY = canvasHeight / 2; // Bird's Y Coord

  // Pipe Info
  const pipeSpeed = 2;
  const pipeWidth = 50;
  const pipeGap = 200;
  let pipeX = 400;
  let topPipeBottomY = canvasHeight / 2;

  // background
  const bg = new Image();
  const pipeUp = new Image();
  const pipeDown = new Image();

  onMount(() => {
    // prepare image
    bg.src = "bg.png";
    bird.src = "bird.png";
    pipeUp.src = "pipe_up.png";
    pipeDown.src = "pipe_down.png";

    // prepare audio
    birdFlySound.src = "bird_fly.mp3";
    score.sound.src = "score.mp3";

    context = canvas.getContext("2d");

    frame = requestAnimationFrame(loop);

    return () => {
      cancelAnimationFrame(frame);
    };
  });

  function loop(dt) {
    frame = requestAnimationFrame(loop);

    // setup background
    context.drawImage(bg, 0, 0, canvasWidth, canvasHeight);

    // setup bird
    context.drawImage(bird, birdX, birdY, birdSize, birdSize);
    updateBird(); // update frame bird

    // setup pipe
    context.drawImage(pipeDown, pipeX, 0, pipeWidth, topPipeBottomY);
    context.drawImage(
      pipeUp,
      pipeX,
      topPipeBottomY + pipeGap,
      pipeWidth,
      canvasHeight
    );
    updatePipe(); // update frame pipe

    // game over and reset
    gameOver() && reset();
  }

  function updateBird() {
    birdY = birdY - birdDy;
    birdDy = birdDy - 0.5;
  }

  function updatePipe() {
    pipeX = pipeX - pipeSpeed;

    if (pipeX == birdX) {
      score.current++;
      score.sound.play();
    }

    if (pipeX < -pipeWidth) {
      pipeX = canvasWidth;
      topPipeBottomY = pipeGap * Math.random();
    }
  }

  function gameOver() {
    // out of canvas
    if (birdY > canvasHeight || birdY < -(birdSize / 2)) return true;

    return (
      (birdY < topPipeBottomY || birdY > topPipeBottomY + pipeGap) &&
      pipeX < birdSize
    );
  }

  function reset() {
    birdDy = 0;
    birdY = canvasHeight / 2 - birdSize;
    pipeX = canvasWidth;
    score.current = 0;
  }

  function handleClick() {
    setBirdSpeed();
    setBirdFlySound();
  }

  function setBirdSpeed() {
    birdDy = birdSpeed;
  }

  function setBirdFlySound() {
    birdFlySound.pause();
    birdFlySound.currentTime = 0;
    birdFlySound.play();
  }
</script>

<div class="container noselect">
  <header>
    <h1>SVELTE FLAPPY BIRD</h1>
    <p>Score: {score.current}</p>
  </header>

  <main on:click={handleClick}>
    <canvas bind:this={canvas} width={canvasWidth} height={canvasHeight} />
  </main>
</div>

<style>
  .container {
    height: 100vh;
  }

  header {
    background-color: aqua;
    font-family: "Teko", sans-serif;
    padding: 20px 0;
    text-align: center;
  }

  main {
    border: 2px dashed #000;
    text-align: center;
  }
</style>
