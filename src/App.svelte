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
  }

  function saveFile() {
    const exportGamesPgn = games
      .map((game) => {
        return makePgn(game);
      })
      .join("\n\n");
    var blob = new Blob([exportGamesPgn], {
      type: "text/plain;charset=utf-8",
    });
    FileSaver.saveAs(blob, "export - " + file[0]["name"]);
  }

  function addClockComment(move: PgnNodeData, clockEntry: string) {
    move.comments = move.comments || [];
    const newClockComment = "[%clk " + parseToPgnTimestamp(clockEntry) + "]";
    const existingClockCommentIndex = move.comments.findIndex((comment) =>
      comment.startsWith("[%clk")
    );

    if (existingClockCommentIndex != -1) {
      if (clockEntry === "") {
        move.comments.splice(existingClockCommentIndex);
      } else {
        move.comments[existingClockCommentIndex] = newClockComment;
      }
    } else if (clockEntry !== "") {
      move.comments.unshift(newClockComment);
    }
  }

  function getClkInMinutes(move: PgnNodeData) {
    const comment = move.comments?.find((comment) =>
      comment.startsWith("[%clk")
    );
    return comment
      ? Number(comment.charAt(6)) * 60 + Number(comment.substring(8, 10))
      : "";
  }

  $: file ? loadGames() : null;

  $: if (games != null) {
    LichessPgnViewer(
      document.getElementsByClassName("viewer")[0] as HTMLElement,
      {
        pgn: makePgn(games[selectedGame]),
      }
    );
  }
</script>

<main>
  <div class="content">
    <div style="width:250px">
      <div style="display:inline; padding: 5px">
        <label for="import" class="btn">Import</label>
        {#if games}
          <label for="export" class="btn">Export</label>
        {/if}
        <input
          id="import"
          class="fileButton"
          accept=".pgn"
          type="file"
          bind:files={file}
        />
        {#if games}
          <input
            id="export"
            class="fileButton"
            type="button"
            on:click={saveFile}
          />
        {/if}
      </div>
      {#if games}
        <div style="margin: 20px 0 0 0">
          {#each games as game, i}
            <button
              class="gameSelector"
              class:currentGameSelected={selectedGame == i}
              value={i}
              on:click={() => {
                selectedGame = i;
              }}
            >
              {game.headers.get("White")} vs {game.headers.get("Black")}
            </button>
          {/each}
        </div>
      {/if}
    </div>
    <div class="viewer" />
    {#if games}
      {#key selectedGame}
        <div class="timeDiv">
          {#each [...games[selectedGame].moves.mainline()] as move, i}
            {#if i % 2 == 0}
              <span class="timeEntryNumber">{i === 0 ? 1 : i / 2 + 1}</span>
            {/if}
            <span class="timeEntryDescriptor">{move.san}:</span>
            <input
              class="timeEntry"
              on:input={(e) => {
                addClockComment(move, e.currentTarget.value);
              }}
              value={getClkInMinutes(move)}
            />
          {/each}
        </div>
      {/key}
    {/if}
  </div>
</main>

<style>
</style>
