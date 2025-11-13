<script lang="ts">
  import { onMount, onDestroy } from "svelte";

  let scores: Array<{ id: string; name: string; score: number } | null> =
    Array(15).fill(null);
  let currentPage = 0;
  let maxPage = 0;
  let eventSource: EventSource | null = null;

  onMount(() => {
    // Create EventSource connection to api.pomomon.club
    eventSource = new EventSource(
      "https://api.pomomon.club/api/leaderboard?page=0"
    );

    eventSource.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);

        // Update leaderboard data from SSE stream
        currentPage = data.page;
        maxPage = data.max_page;

        // Ensure we have exactly 15 slots
        const newScores = Array(15).fill(null);
        data.scores.forEach(
          (
            score: { id: string; name: string; score: number },
            index: number
          ) => {
            if (index < 15) {
              newScores[index] = score;
            }
          }
        );

        scores = newScores;
      } catch (error) {
        console.error("Error parsing SSE data:", error);
      }
    };

    eventSource.onerror = (error) => {
      console.error("EventSource error:", error);
      // EventSource will automatically attempt to reconnect
    };
  });

  onDestroy(() => {
    // Clean up EventSource connection when component is destroyed
    if (eventSource) {
      eventSource.close();
    }
  });

  function formatTime(milliseconds: number): string {
    const totalSeconds = milliseconds / 1000;
    const totalMinutes = totalSeconds / 60;
    const totalHours = totalMinutes / 60;

    if (totalHours >= 1) {
      return `${totalHours.toFixed(1)}h`;
    } else if (totalMinutes >= 1) {
      return `${totalMinutes.toFixed(1)}min`;
    } else {
      return `${totalSeconds.toFixed(1)}s`;
    }
  }
</script>

<div class="flex items-center justify-center">
  <div class="flex flex-col gap-3 w-full font-mono">
    <div class="flex flex-col gap-2 w-full">
      {#each scores as entry, index}
        <div
          class="flex items-center justify-between p-4 w-full bg-bg-700 rounded-lg border border-bg-600 hover:border-bg-500 transition-colors"
        >
          <div class="flex items-center gap-4">
            <span class="text-bg-300 font-bold text-lg w-8">#{index + 1}</span>
            {#if entry}
              <span class="text-bg-150 text-lg">{entry.name}</span>
            {:else}
              <span class="text-bg-400 italic">no score yet</span>
            {/if}
          </div>
          <div>
            {#if entry}
              <span class="text-p-blue-200 font-bold text-lg"
                >{formatTime(entry.score)}</span
              >
            {/if}
          </div>
        </div>
      {/each}
    </div>
  </div>
</div>
