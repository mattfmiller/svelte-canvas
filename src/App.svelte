<script lang="ts">
  import { onMount } from "svelte";
  import Canvas1 from "./components/Canvas1.svelte";

  const src = "assets/neg2.jpg";
  const width = 900;
  const height = 600;
  let image: HTMLImageElement;

  onMount(async () => {
    image = await loadImage(src);
  });

  async function loadImage(imagePath: string): Promise<HTMLImageElement> {
    return new Promise((resolve, reject) => {
      const image = new Image();
      image.src = imagePath;
      image.addEventListener("load", () => {
        resolve(image);
      });
      image.addEventListener("error", (err) => {
        reject(err);
      });
    });
  }
</script>

<main>
  <div>
    <img alt="some dumb pic" {src} {width} {height} />
  </div>
  {#if image}
    <Canvas1 {image} />
  {/if}
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
