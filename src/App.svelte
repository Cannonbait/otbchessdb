<script lang="ts">
  import { pgnRead, pgnWrite, Database, Game, type GamePOJO } from "kokopu";
  import { range } from "lodash";
  import exporter from "@chessclub.com/chesspgn/app/exporter";
  import Importer from "@chessclub.com/chesspgn/app/importer";
  import {
    intervalToDuration,
    addMinutes,
    startOfDay,
    parse as parseDate,
  } from "date-fns";
  import LichessPgnViewer from "lichess-pgn-viewer";
  import { each } from "svelte/internal";
  import "./lichess-pgn-viewer.css";

  const parse = (pgn: string) => pgnRead(pgn);

  let file;
  let reader: Database;
  let jsonGames: GamePOJO[];
  let selectedGame: number;
  let element;
  let rawGames: string[];

  function parseToPgnTimestamp(str: string) {
    let d = startOfDay(new Date());
    let ourDate = addMinutes(d, Number(str));
    let int = intervalToDuration({ start: d, end: ourDate });

    return (
      int.hours +
      ":" +
      (int.minutes > 9 ? int.minutes : "0" + int.minutes) +
      ":00"
    );
  }

  async function loadGames() {
    let loadedFile = await file[0].text();
    reader = parse(loadedFile);
    jsonGames = range(reader.gameCount()).map((i) => reader.game(i).pojo());
    rawGames = loadedFile.trim().split("\n\n\n");
  }

  function saveFile() {
    console.log(pgnWrite(jsonGames.map((g) => Game.fromPOJO(g))));
  }

  $: file ? loadGames() : null;
  $: console.log(jsonGames ? jsonGames[selectedGame] : null);
  $: element
    ? console.log("Element: " + element)
    : console.log("Lost viewer div");

  $: {
    console.log("Selected game thing triggered");
    console.log(selectedGame);
    if (selectedGame != null) {
      console.log("Setting lpv to pgn: " + rawGames[selectedGame]);
      console.log(document.getElementsByClassName("viewer")[0] as HTMLElement);
      const element = document.getElementsByClassName(
        "viewer"
      )[0] as HTMLElement;
      LichessPgnViewer(element, {
        pgn: rawGames[selectedGame],
      });
    }
  }
</script>

<main>
  Select a pgn file
  <input accept=".pgn" type="file" bind:files={file} />
  {#if reader}
    <div>Games loaded</div>
    <form>
      <select bind:value={selectedGame}>
        {#each jsonGames as game, i}
          <option value={i}>
            {game.white.name} vs {game.black.name}
          </option>
        {/each}
      </select>
    </form>
  {/if}
  <div class="content">
    <div class="viewer" />
    <!-- {#if selectedGame != null}
      <div class="timeThing">
        {#each jsonGames[selectedGame].mainVariation as move}
          <span style="text-align:right">{move.notation}:</span>
          <input
            placeholder={move.notation}
            on:input={(e) => {
              move.tags.clk = parseToPgnTimestamp(e.currentTarget.value);
            }}
          />
        {/each}
      </div>
    {/if} -->
  </div>
  <button on:click={saveFile}>Save file</button>
</main>

<style>
  .timeThing {
    display: grid;
    grid-template-columns: repeat(4, 100px);
  }
  .content {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 25px;
  }
</style>
