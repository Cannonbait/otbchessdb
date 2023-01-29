<script lang="ts">
  import { range } from "lodash";
  import { intervalToDuration, addMinutes, startOfDay } from "date-fns";
  import LichessPgnViewer from "lichess-pgn-viewer";
  import "./lichess-pgn-viewer.css";
  import { parsePgn, makePgn, type PgnNodeData, type Game } from "chessops/pgn";
  import { saveAs } from "file-saver";
  import * as FileSaver from "file-saver";

  let file: FileList;
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
    console.log(file);
  }

  function saveFile() {
    const exportGamesPgn = games
      .map((game) => {
        return makePgn(game);
      })
      .join("\n\n");
    console.log(exportGamesPgn);
    var blob = new Blob([exportGamesPgn], {
      type: "text/plain;charset=utf-8",
    });
    FileSaver.saveAs(blob, "export - " + file[0]["name"]);
  }

  function addClockComment(move: PgnNodeData, clockEntry: string) {
    const newClockComment = "[%clk " + parseToPgnTimestamp(clockEntry) + "]";
    console.log(move);
    const existingClockCommentIndex = move.comments.findIndex((comment) =>
      comment.startsWith("[%clk")
    );
    if (existingClockCommentIndex != -1) {
      move.comments[existingClockCommentIndex] = newClockComment;
    } else {
      move.comments.unshift(newClockComment);
    }
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
  <input accept=".pgn" type="file" bind:files={file} />
  {#if games}
    <button on:click={saveFile}>Save file</button>
  {/if}

  <div class="content">
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
    <div class="viewer" />
    {#if games}
      <div class="timeThing">
        {#each Array.from(games[selectedGame].moves.mainline()) as move}
          <span style="text-align:right">{move.san}:</span>
          <input
            on:input={(e) => {
              addClockComment(move, e.currentTarget.value);
            }}
          />
        {/each}
      </div>
    {/if}
  </div>
</main>

<style>
  .timeThing {
    display: grid;
    grid-template-columns: repeat(4, 50px);
  }
  .content {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    gap: 25px;
  }
</style>
