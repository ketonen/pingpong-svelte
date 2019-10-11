<script>
  export let ball;

  // $: () => {
  //   document.getElementById('own').left
  //   document.getElementById('enemy').left
  // }
</script>

<style>
  #ball {
    height: 25px;
    width: 25px;
    background-color: black;
    position: absolute;
    border-radius: 50%;
    display: inline-block;
  }
</style>

<span id="ball" style="left:{ball.x}px;top:{ball.y}px" />
