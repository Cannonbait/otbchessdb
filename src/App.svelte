<script lang="ts">
  import { parse, type ParseTree } from "@mliebelt/pgn-parser";
  import { writeGame } from "@mliebelt/pgn-writer";
  import LichessPgnViewer from "lichess-pgn-viewer";
  import "./lichess-pgn-viewer.css";

  interface GamesType {
    rawData: string;
    parsed: ParseTree;
  }

  let file;
  let games: GamesType[];
  let selectedGame: GamesType;
  let lpv;

  async function loadGames() {
    games = (await file[0].text())
      .trim()
      .split("\n\n\n")
      .map((string) => {
        console.log(string);
        return {
          rawData: string,
          parsed: parse(string, { startRule: "game" }),
        };
      });
  }

  function handleGameSelected() {}

  $: {
    if (file != null) {
      loadGames();
    }
  }
  $: console.log(selectedGame);
  $: {
    if (selectedGame != null) {
      console.log(selectedGame);
      lpv = LichessPgnViewer(
        document.getElementsByClassName("viewer")[0] as HTMLElement,
        {
          pgn: selectedGame.rawData,
        }
      );
    }
  }
</script>

<main>
  Select a pgn file
  <input accept=".pgn" type="file" bind:files={file} />
  {#if games}
    <div>Games loaded</div>
    <form on:submit|preventDefault={handleGameSelected}>
      <select bind:value={selectedGame}>
        {#each games as game}
          <option value={game}>
            {game.parsed.tags.White} vs {game.parsed.tags.Black}
          </option>
        {/each}
      </select>
    </form>
  {/if}
  <div class="viewer" />
</main>

<style>
</style>
