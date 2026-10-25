<script lang="ts">
  import { Buffer } from "buffer";
  import { gzipSync } from "fflate";
  import { gameData } from "../store";
  import Button from "./components/Button.svelte";

  const download = () => {
    let saveFile;

    const unsubscribe = gameData.subscribe((value) => {
      saveFile = value;
    });
    unsubscribe();

    if (!saveFile) {
      console.error("No save file loaded.");
      return;
    }

    try {
      const jsonString = JSON.stringify(saveFile).replaceAll("class_", "class ");
      const buffer: Uint8Array = Buffer.from(jsonString, "utf8");
      const data = gzipSync(buffer);
      const blob = new Blob([data], { type: "application/octet-stream" });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");

      a.href = url;
      a.download = "game-edited.dat";
      document.body.appendChild(a);
      a.click();
      a.remove();
      setTimeout(() => window.URL.revokeObjectURL(url), 1000);
    } catch (error) {
      console.error("Failed to export save:", error);
    }
  };
</script>

{#if $gameData}
  <div>
    <Button on:click={download}>Download edited save</Button>
  </div>
{/if}

