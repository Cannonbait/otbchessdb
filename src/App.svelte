<script lang="ts">
  import { range } from "lodash";
  import { intervalToDuration, addMinutes, startOfDay } from "date-fns";
  import LichessPgnViewer from "lichess-pgn-viewer";
  import "./lichess-pgn-viewer.css";
  import { parsePgn, makePgn, type PgnNodeData, type Game } from "chessops/pgn";

  let file;
  let games: Game<PgnNodeData>[];
  let selectedGame: number = 0;

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
    let fileText = await file[0].text();
    games = parsePgn(fileText);
  }

  function saveFile() {
    const exportGamesPgn = games
      .map((game) => {
        return makePgn(game);
      })
      .join("\n\n");
    console.log(exportGamesPgn);
  }

  $: file ? loadGames() : null;

  $: {
    if (games != null) {
      LichessPgnViewer(
        document.getElementsByClassName("viewer")[0] as HTMLElement,
        {
          pgn: makePgn(games[selectedGame]),
        }
      );
    }
  }
</script>

<main>
  Select a pgn file
  <input accept=".pgn" type="file" bind:files={file} />
  {#if games}
    <form>
      <select bind:value={selectedGame}>
        {#each games as game, i}
          <option value={i}>
            {game.headers.get("White")} vs {game.headers.get("Black")}
          </option>
        {/each}
      </select>
    </form>
  {/if}

  <div class="content">
    <div class="viewer" />
    {#if games}
      <div class="timeThing">
        {#each Array.from(games[selectedGame].moves.mainline()) as move}
          <span style="text-align:right">{move.san}:</span>
          <input
            on:input={(e) => {
              console.log(parseToPgnTimestamp(e.currentTarget.value));
            }}
          />
        {/each}
      </div>
    {/if}
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
