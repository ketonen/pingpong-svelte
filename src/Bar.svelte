<script>
  export let type;
  export let position;  
</script>

<style>
  div {
    min-height: 10px;
    max-width: 200px;
    position: fixed;
    width: 100%;
  }
  div#own {
    background-color: black;
    position: fixed;
    bottom: 1%;
  }
  div#enemy {
    top: 1%;
    background-color: red;
  }
</style>

<div id={type} style="left:{position}%" />
