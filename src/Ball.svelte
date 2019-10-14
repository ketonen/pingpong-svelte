<script>
  import { spring } from "svelte/motion";
  export let coords;
  let size = spring(10);
</script>

<style>
  svg {
    width: 100%;
    height: 100%;
  }
  circle {
    fill: black;
  }
</style>

<svg>
  <circle id="ball" cx={$coords.x} cy={$coords.y} r={$size} />
</svg>
