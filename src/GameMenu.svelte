<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
</script>

<style>
  .modal-background {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.3);
  }

  .modal {
    position: absolute;
    left: 50%;
    top: 50%;
    width: calc(100vw - 4em);
    max-width: 32em;
    max-height: calc(100vh - 4em);
    overflow: auto;
    transform: translate(-50%, -50%);
    padding: 1em;
    border-radius: 0.2em;
    background: white;
  }

  button {
    display: block;
    font-size: 20px;
  }
</style>

<div class="modal-background" on:click={() => dispatch('close')} />

<div class="modal">
  <slot name="header" />
  <hr />
  <button id="local" on:click={() => dispatch('gameModeSelected', 'local')}>Local game</button>
  <button id="online" on:click={() => dispatch('gameModeSelected', 'online')}>Online game</button>
  <hr />
</div>
