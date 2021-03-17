<script lang="ts">
  import { onMount } from "svelte";

  export let image: HTMLImageElement;

  const width = 600;
  const height = 400;

  let canvas1: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let brightness = 0;
  let imageData;

  onMount(() => {
    ctx = canvas1.getContext("2d")!;
    ctx.drawImage(image, 0, 0, image.width, image.height, 0, 0, width, height);
    imageData = ctx.getImageData(0, 0, width, height);
  });

  function handleBrightness() {
    const adjust = Math.floor(255 * (brightness / 100));
    ctx.drawImage(image, 0, 0, image.width, image.height, 0, 0, width, height);
    // const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    for (var i = 0; i < data.length; i += 4) {
      data[i] += adjust; // red
      data[i + 1] += adjust; // green
      data[i + 2] += adjust; // blue
    }
    ctx.putImageData(imageData, 0, 0);
  }

  function invert() {
    ctx.drawImage(image, 0, 0, image.width, image.height, 0, 0, width, height);
    const data = imageData.data;
    for (var i = 0; i < data.length; i += 4) {
      data[i] = 255 - data[i]; // red
      data[i + 1] = 255 - data[i + 1]; // green
      data[i + 2] = 255 - data[i + 2]; // blue
    }
    ctx.putImageData(imageData, 0, 0);
  }
</script>

<div>
  <canvas bind:this={canvas1} {width} {height} />
  <div>
    <button on:click={invert}>invert</button>
  </div>
  <div class="flex">
    <span>brightness</span>
    <input
      bind:value={brightness}
      type="range"
      min="-100"
      max="100"
      class="slider"
      on:change={handleBrightness}
    />
    <input
      value={brightness}
      type="number"
      min="-100"
      max="100"
      on:change={handleBrightness}
    />
  </div>
</div>

<style>
  canvas {
    border: 1px solid black;
    background-color: darkgray;
  }
  .flex {
    display: "flex";
  }
</style>
