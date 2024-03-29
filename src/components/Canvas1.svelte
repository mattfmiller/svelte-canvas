<script lang="ts">
  import { onMount } from "svelte";
  import Slider from "./Slider.svelte";

  export let image: HTMLImageElement;

  const width = 900;
  const height = 600;

  let canvas1: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let brightness = 0;
  let exposure = 0;
  let contrast = 0;
  let colorTemperature = 0;
  let tint = 0;
  let saturation = 0;
  let vibrance = 0;
  let sharpen = 0;
  let originalImageData: ImageData;
  let shouldInvert: boolean;
  let shouldCorrectColors: boolean;

  onMount(() => {
    ctx = canvas1.getContext("2d")!;
    ctx.drawImage(image, 0, 0, image.width, image.height, 0, 0, width, height);
    originalImageData = ctx.getImageData(0, 0, width, height);
  });

  function nonZeroMin(numbers: number[]): number {
    const nonZeroNumbers = [...new Set(numbers.filter((n) => n > 0))];
    return Math.min(...nonZeroNumbers);
  }

  function handleBrightness(value) {
    brightness = value;
    applyFilters();
  }

  function handleExposure(value) {
    exposure = value;
    applyFilters();
  }

  function handleContrast(value) {
    contrast = value;
    applyFilters();
  }

  function handleColorTemperature(value) {
    colorTemperature = value;
    applyFilters();
  }

  function handleTint(value) {
    tint = value;
    applyFilters();
  }

  function handleSaturation(value) {
    saturation = value;
    applyFilters();
  }

  function handleVibrance(value) {
    vibrance = value;
    applyFilters();
  }

  function handleSharpen(value) {
    sharpen = value;
    applyFilters();
  }

  function invert() {
    shouldInvert = !shouldInvert;
    applyFilters();
  }

  function correctColors() {
    shouldCorrectColors = !shouldCorrectColors;
    applyFilters();
  }

  function applyInvert(data: Uint8ClampedArray) {
    for (let i = 0; i < data.length; i += 4) {
      data[i] = 255 - data[i]; // red
      data[i + 1] = 255 - data[i + 1]; // green
      data[i + 2] = 255 - data[i + 2]; // blue
    }
    return data;
  }

  function applyColorCorrection(data: Uint8ClampedArray) {
    const redData: number[] = [];
    const greenData: number[] = [];
    const blueData: number[] = [];
    for (let i = 0; i < data.length; i += 4) {
      redData.push(data[i]); // red
      greenData.push(data[i + 1]); // green
      blueData.push(data[i + 2]); // blue
    }

    const minRed = nonZeroMin(redData);
    const minGreen = nonZeroMin(greenData);
    const minBlue = nonZeroMin(blueData);
    const maxRed = Math.max(...[...new Set(redData)]);
    const maxGreen = Math.max(...[...new Set(greenData)]);
    const maxBlue = Math.max(...[...new Set(blueData)]);
    const redSlope = 255 / (maxRed - minRed);
    const greenSlope = 255 / (maxGreen - minGreen);
    const blueSlope = 255 / (maxBlue - minBlue);
    const redIntercept = -1 * redSlope * minRed;
    const greenIntercept = -1 * greenSlope * minGreen;
    const blueIntercept = -1 * blueSlope * minBlue;

    for (let i = 0; i < data.length; i += 4) {
      data[i] = redSlope * data[i] + redIntercept; // red
      data[i + 1] = greenSlope * data[i + 1] + greenIntercept; // green
      data[i + 2] = blueSlope * data[i + 2] + blueIntercept; // blue
    }

    return data;
  }

  function applyBrightness(data: Uint8ClampedArray) {
    const adjust = Math.floor(255 * (brightness / 100));
    for (let i = 0; i < data.length; i += 4) {
      data[i] += adjust; // red
      data[i + 1] += adjust; // green
      data[i + 2] += adjust; // blue
    }
    return data;
  }

  // use curves helper eventually
  function applyExposure(data: Uint8ClampedArray) {
    const adjust = exposure;
    // for (let i = 0; i < data.length; i += 4) {
    //   data[i] += adjust; // red
    //   data[i + 1] += adjust; // green
    //   data[i + 2] += adjust; // blue
    // }
    return data;
  }

  // caman filter
  // function applyContrast(data: Uint8ClampedArray) {
  //   const adjust = Math.pow((contrast + 100) / 100, 2);
  //   const filter = (data: number): number =>
  //     ((data / 255 - 0.5) * adjust + 0.5) * 255;
  //   for (let i = 0; i < data.length; i += 4) {
  //     data[i] = filter(data[i]); // red
  //     data[i + 1] = filter(data[i + 1]); // green
  //     data[i + 2] = filter(data[i + 2]); // blue
  //   }
  //   return data;
  // }

  // alternate filter
  // use curves eventually?
  function applyContrast(data: Uint8ClampedArray) {
    const adjust = contrast / 100 + 1; //convert to decimal & shift range: [0..2];
    const intercept = 128 * (1 - adjust);
    for (let i = 0; i < data.length; i += 4) {
      data[i] = data[i] * adjust + intercept; // red
      data[i + 1] = data[i + 1] * adjust + intercept; // green
      data[i + 2] = data[i + 2] * adjust + intercept; // blue
    }
    return data;
  }

  function applyColorTemperature(data: Uint8ClampedArray) {
    const adjust = colorTemperature;
    for (let i = 0; i < data.length; i += 4) {
      data[i] += adjust; // red
      data[i + 2] -= adjust; // blue
    }
    return data;
  }

  function applyTint(data: Uint8ClampedArray) {
    const adjust = tint;
    for (let i = 0; i < data.length; i += 4) {
      data[i + 1] += adjust; // green
    }
    return data;
  }

  function applySaturation(data: Uint8ClampedArray) {
    const adjust = -0.01 * saturation;
    for (let i = 0; i < data.length; i += 4) {
      const max = Math.max(data[i], data[i + 1], data[i + 2]);
      data[i] += data[i] === max ? 0 : (max - data[i]) * adjust; // red
      data[i + 1] += data[i + 1] === max ? 0 : (max - data[i + 1]) * adjust; // green
      data[i + 2] += data[i + 2] === max ? 0 : (max - data[i + 2]) * adjust; // blue
    }
    return data;
  }

  // weak?
  function applyVibrance(data: Uint8ClampedArray) {
    const adjust = -1 * vibrance;
    for (let i = 0; i < data.length; i += 4) {
      const max = Math.max(data[i], data[i + 1], data[i + 2]);
      const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
      const amt = (((Math.abs(max - avg) * 2) / 255) * adjust) / 100;
      data[i] += data[i] === max ? 0 : (max - data[i]) * amt; // red
      data[i + 1] += data[i + 1] === max ? 0 : (max - data[i + 1]) * amt; // green
      data[i + 2] += data[i + 2] === max ? 0 : (max - data[i + 2]) * amt; // blue
    }
    return data;
  }

  // refactor loops? Needs refinement?
  function applySharpen(data: Uint8ClampedArray, w: number, h: number) {
    const adjust = sharpen / 100 / 2;
    const weights = [
      0,
      -adjust,
      0,
      -adjust,
      4 * adjust + 1,
      -adjust,
      0,
      -adjust,
      0,
    ];
    const side = Math.round(Math.sqrt(weights.length));
    const halfSide = Math.floor(side / 2);
    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        const sy = y;
        const sx = x;
        const dstOff = (y * w + x) * 4;
        // calculate the weighed sum of the source image pixels that
        // fall under the convolution matrix
        let r = 0;
        let g = 0;
        let b = 0;
        for (var cy = 0; cy < side; cy++) {
          for (var cx = 0; cx < side; cx++) {
            var scy = sy + cy - halfSide;
            var scx = sx + cx - halfSide;
            if (scy >= 0 && scy < h && scx >= 0 && scx < w) {
              var srcOff = (scy * w + scx) * 4;
              var wt = weights[cy * side + cx];
              r += data[srcOff] * wt;
              g += data[srcOff + 1] * wt;
              b += data[srcOff + 2] * wt;
            }
          }
        }
        data[dstOff] = r;
        data[dstOff + 1] = g;
        data[dstOff + 2] = b;
      }
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
    if (shouldCorrectColors) {
      data = applyColorCorrection(data);
    }
    data = applyBrightness(data);
    data = applyExposure(data);
    data = applyContrast(data);
    data = applyColorTemperature(data);
    data = applyTint(data);
    data = applySaturation(data);
    data = applyVibrance(data);
    data = applySharpen(data, width, height);

    ctx.putImageData(imageData, 0, 0);
  }
</script>

<div>
  <canvas bind:this={canvas1} {width} {height} />
  <div>
    <button on:click={invert}>invert</button>
    <button on:click={correctColors}>normalize colors</button>
  </div>
  <Slider name="brightness" callback={handleBrightness} />
  <Slider name="exposure" callback={handleExposure} />
  <Slider name="contrast" callback={handleContrast} />
  <Slider name="temperature" callback={handleColorTemperature} />
  <Slider name="tint" callback={handleTint} />
  <Slider name="vibrance" callback={handleVibrance} />
  <Slider name="saturation" callback={handleSaturation} />
  <Slider name="sharpen" callback={handleSharpen} />
</div>

<style>
  canvas {
    border: 1px solid black;
    background-color: darkgray;
  }
  /* .flex {
    display: "flex";
  } */
</style>
