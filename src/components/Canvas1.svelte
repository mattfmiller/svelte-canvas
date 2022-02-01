<script lang="ts">
  import { onMount } from "svelte";

  export let image: HTMLImageElement;

  const width = 600;
  const height = 400;

  let canvas1: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let brightness = 0;
  let originalImageData: ImageData;
  let shouldInvert: boolean;

  onMount(() => {
    ctx = canvas1.getContext("2d")!;
    ctx.drawImage(image, 0, 0, image.width, image.height, 0, 0, width, height);
    originalImageData = ctx.getImageData(0, 0, width, height);
  });

  function handleBrightness() {
    applyFilters();
  }

  function invert() {
    shouldInvert = !shouldInvert;
    applyFilters();
  }

  function applyInvert(data: Uint8ClampedArray) {
    for (var i = 0; i < data.length; i += 4) {
      data[i] = 255 - data[i]; // red
      data[i + 1] = 255 - data[i + 1]; // green
      data[i + 2] = 255 - data[i + 2]; // blue
    }
    return data;
  }

  function applyBrightness(data: Uint8ClampedArray) {
    const adjust = Math.floor(255 * (brightness / 100));
    for (var i = 0; i < data.length; i += 4) {
      data[i] += adjust; // red
      data[i + 1] += adjust; // green
      data[i + 2] += adjust; // blue
    }
    return data;
  }

  function applyFilters() {
    ctx.drawImage(image, 0, 0, image.width, image.height, 0, 0, width, height);
    const imageData = ctx.getImageData(0, 0, width, height);
    let data = imageData.data;
    if (shouldInvert) {
      data = applyInvert(data);
    }
    data = applyBrightness(data);

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
      on:change={applyFilters}
    />
    <input
      value={brightness}
      type="number"
      min="-100"
      max="100"
      on:change={applyFilters}
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
