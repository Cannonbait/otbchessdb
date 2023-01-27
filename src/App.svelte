<script lang="ts">
  async function printToConsole() {
    console.log(await file[0].text());
  }
  let file;
  let textFile;
  $: {
    if (file != null) {
      printToConsole();
    }
  }

  async function doTheConsoleLog() {}
</script>

<main>
  Select a pgn file
  <input accept=".pgn" type="file" bind:files={file} />
  {#if file}
    <div>File selected</div>
  {/if}
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
