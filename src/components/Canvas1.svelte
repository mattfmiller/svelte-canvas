<script lang="ts">
  import { onMount } from "svelte";

  export let image: HTMLImageElement;

  const width = 600;
  const height = 400;

  let canvas1: HTMLCanvasElement;
  let canvasContext: CanvasRenderingContext2D;

  onMount(() => {
    canvasContext = canvas1.getContext("2d")!;
    canvasContext.drawImage(
      image,
      0,
      0,
      image.width,
      image.height,
      0,
      0,
      600,
      400
    );
  });
</script>

<canvas bind:this={canvas1} {width} {height} />

<style>
  canvas {
    border: 1px solid black;
    background-color: darkgray;
  }
</style>
