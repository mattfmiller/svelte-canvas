<script lang="ts">
  export let name: string;
  export let callback: (v: number) => void;

  let value: number = 0;

  const handleChange = (v) => {
    callback(value);
  };
</script>

<div class="flex">
  <span>{name}</span>
  <input
    bind:value
    type="range"
    min="-100"
    max="100"
    class="slider"
    on:change={handleChange}
  />
  <input {value} type="number" min="-100" max="100" on:change={handleChange} />
</div>

<style>
  .flex {
    display: "flex";
  }
</style>
