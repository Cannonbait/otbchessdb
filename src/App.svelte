<script lang="ts">
  import { type ParseTree } from "@mliebelt/pgn-parser";
  import { writeGame } from "@mliebelt/pgn-writer";
  import { PgnReader } from "@mliebelt/pgn-reader";
  import {
    intervalToDuration,
    addMinutes,
    startOfDay,
    parse as parseDate,
  } from "date-fns";
  import LichessPgnViewer from "lichess-pgn-viewer";
  import { each } from "svelte/internal";
  import "./lichess-pgn-viewer.css";

  let rawDataString = `[Event "Rated Blitz game"]
[Site "https://lichess.org/R05ZIxG4"]
[Date "2022.11.24"]
[White "nagarajesh"]
[Black "Cannonbait"]
[Result "1-0"]
[UTCDate "2022.11.24"]
[UTCTime "17:01:35"]
[WhiteElo "1817"]
[BlackElo "1820"]
[WhiteRatingDiff "+6"]
[BlackRatingDiff "-25"]
[Variant "Standard"]
[TimeControl "300+3"]
[ECO "C52"]
[Opening "Italian Game: Evans Gambit, Tartakower Attack"]
[Termination "Time forfeit"]
[Annotator "lichess.org"]

1. e4 { [%clk 0:05:00] } 1... e5 { [%clk 0:05:00] } 2. Nf3 { [%clk 0:05:01] } 2... Nc6 { [%clk 0:04:57] } 3. Bc4 { [%clk 0:05:03] } 3... Bc5 { [%clk 0:04:55] } 4. b4 { [%clk 0:05:04] } 4... Bxb4 { [%clk 0:04:56] } 5. c3 { [%clk 0:05:07] } 5... Ba5 { [%clk 0:04:58] } 6. d4 { [%clk 0:05:09] } 6... d6 { [%clk 0:04:56] } 7. Qb3 { [%clk 0:05:06] } { C52 Italian Game: Evans Gambit, Tartakower Attack } 7... Qe7 { [%clk 0:04:46] } 8. O-O { [%clk 0:05:04] } 8... Nf6 { [%clk 0:04:46] } 9. dxe5 { [%clk 0:05:06] } 9... dxe5 { [%clk 0:04:46] } 10. Na3 { [%clk 0:05:00] } 10... O-O { [%clk 0:04:40] } 11. Nc2 { [%clk 0:04:58] } 11... Bb6 { [%clk 0:04:32] } 12. Ne3 { [%clk 0:04:49] } 12... Na5 { [%clk 0:04:33] } 13. Qa4 { [%clk 0:04:39] } 13... Bd7 { [%clk 0:04:25] } 14. Bb5 { [%clk 0:04:36] } 14... Bxb5 { [%clk 0:04:17] } 15. Qxb5 { [%clk 0:04:38] } 15... Nxe4 { [%clk 0:03:55] } 16. c4 { [%clk 0:04:27] } 16... a6 { [%clk 0:03:37] } 17. Qb1 { [%clk 0:04:24] } 17... Nc3 { [%clk 0:03:34] } 18. Qf5 { [%clk 0:04:15] } 18... Ne2+ { [%clk 0:03:32] } 19. Kh1 { [%clk 0:04:15] } 19... Nxc1 { [%clk 0:03:18] } 20. Nd5 { [%clk 0:04:09] } 20... Qc5 { [%clk 0:02:55] } 21. Raxc1 { [%clk 0:04:05] } 21... Nxc4 { [%clk 0:02:43] } 22. Ne3 { [%clk 0:04:05] } 22... Nxe3 { [%clk 0:02:35] } 23. fxe3 { [%clk 0:04:07] } 23... Qxe3 { [%clk 0:02:34] } 24. Nxe5 { [%clk 0:04:08] } 24... f6 { [%clk 0:02:01] } 25. Qe6+ { [%clk 0:04:07] } 25... Kh8 { [%clk 0:01:55] } 26. Nf7+ { [%clk 0:04:08] } 26... Rxf7 { [%clk 0:01:57] } 27. Qxf7 { [%clk 0:04:10] } 27... h6 { [%clk 0:01:31] } 28. Rfe1 { [%clk 0:04:09] } 28... Qd2 { [%clk 0:01:30] } 29. Rcd1 { [%clk 0:04:09] } 29... Qf2 { [%clk 0:01:05] } 30. Re7 { [%clk 0:04:00] } 30... Rg8 { [%clk 0:00:30] } 31. h3 { [%clk 0:03:59] } 31... Qg3 { [%clk 0:00:31] } 32. Rf1 { [%clk 0:03:51] } 32... Qc3 { [%clk 0:00:18] } 33. Rd7 { [%clk 0:03:45] } 33... Qc5 { [%clk 0:00:07] } 34. Rfd1 { [%clk 0:03:37] } 34... Qe5 { [%clk 0:00:05] } 35. Re7 { [%clk 0:03:37] } 35... Qg5 { [%clk 0:00:05] } 36. Rf1 { [%clk 0:03:35] } 36... Qg3 { [%clk 0:00:07] } 37. Rxf6 { [%clk 0:03:32] } { White wins on time. } 1-0


`;

  const parse = (pgn: string) => new PgnReader({ pgn });

  let file;
  let reader: PgnReader;
  let selectedGame: number;
  let lpv;
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
    rawGames = loadedFile.trim().split("\n\n\n");
  }

  function saveFile() {
    console.log(reader.writePgn());
  }

  $: {
    if (file != null) {
      loadGames();
    }
  }

  $: {
    if (selectedGame != null && element) {
      lpv = LichessPgnViewer(element, {
        pgn: rawGames[selectedGame],
      });
    }
  }
  $: console.log(selectedGame);
</script>

<main>
  Select a pgn file
  <input accept=".pgn" type="file" bind:files={file} />
  {#if reader}
    <div>Games loaded</div>
    <form>
      <select bind:value={selectedGame}>
        {#each reader.games as game, i}
          <option value={i}>
            {game.tags.White} vs {game.tags.Black}
          </option>
        {/each}
      </select>
    </form>
  {/if}
  <div class="content">
    <div class="viewer" bind:this={element} />
    {#if selectedGame != null}
      <div class="timeThing">
        {#each reader.games[selectedGame].moves as move, i}
          <span style="text-align:right">{move.notation.notation}:</span>
          <input
            placeholder={move.notation.notation}
            on:input={(e) =>
              (move.commentDiag.clk = parseToPgnTimestamp(
                e.currentTarget.value
              ))}
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
