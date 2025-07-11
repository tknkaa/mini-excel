<script lang="ts">
  import { setupPlot } from "$lib/chart";
  import type { Cell } from "$lib/types.ts";
  import {
    Chart,
    ScatterController,
    LinearScale,
    PointElement,
    Tooltip,
    Legend,
  } from "chart.js/auto";

  Chart.register(ScatterController, LinearScale, PointElement, Tooltip, Legend);

  const rows = 10;
  const cols = 10;

  let grid: Cell[][] = $state(
    Array.from({ length: rows }, (_, y) =>
      Array.from({ length: cols }, (_, x) => ({
        x,
        y,
        isSelected: false,
        value: "",
        isWritable: false,
      })),
    ),
  );

  function convertLocationToCell(event: MouseEvent) {
    const target = event.target as HTMLElement;
    const rowElem = target.parentElement;
    if (!rowElem || !rowElem.parentElement) return null;
    const rowIndex = Array.from(rowElem.parentElement.children).indexOf(
      rowElem,
    );
    const cellIndex = Array.from(rowElem.children).indexOf(target);
    return grid[rowIndex][cellIndex];
  }

  let leftTopCell: Cell | null = $state(null);

  function handleMouseDown(event: MouseEvent) {
    grid.forEach((row) => row.forEach((cell) => (cell.isSelected = false)));
    const cell = convertLocationToCell(event);
    if (cell) {
      leftTopCell = cell;
    }
  }

  function handleMouseUp(event: MouseEvent) {
    const cell = convertLocationToCell(event);
    if (cell && leftTopCell) {
      const startX = Math.min(leftTopCell.x, cell.x);
      const endX = Math.max(leftTopCell.x, cell.x);
      const startY = Math.min(leftTopCell.y, cell.y);
      const endY = Math.max(leftTopCell.y, cell.y);

      for (let y = startY; y <= endY; y++) {
        for (let x = startX; x <= endX; x++) {
          grid[y][x].isSelected = true;
        }
      }
    }
  }

  let chartInstance: Chart | null = null;

  function createPlot() {
    const selectedCells = grid.flat().filter((cell) => cell.isSelected);
    if (selectedCells.length % 2 !== 0) {
      alert("Please select an even number of cells.");
      return;
    }
    console.log("Selected cells:", selectedCells);
    const values = selectedCells.map((cell) => parseFloat(cell.value));
    if (values.some((value) => isNaN(value))) {
      alert("Invalid value in selected cells. Please enter valid numbers.");
      return;
    }
    console.log("Selected values:", values);
    const ctx = document.getElementById("chart") as HTMLCanvasElement;
    const config = setupPlot(values);
    if (chartInstance) {
      chartInstance.destroy();
    }
    chartInstance = new Chart(ctx, config);
  }
</script>

<div class="grid">
  {#each grid as row}
    <div class="row">
      {#each row as cell}
        {#if cell.isWritable}
          <input
            type="text"
            class="cell"
            bind:value={cell.value}
            onchange={(event: Event) => {
              cell.value = (event.target as HTMLInputElement).value;
            }}
            onkeydown={(event: KeyboardEvent) => {
              if (event.key === "Enter") {
                cell.isWritable = false;
                cell.isSelected = false;
              }
            }}
          />
        {:else}
          <button
            class="cell {cell.isSelected ? 'selected' : ''}"
            onmousedown={(event) => handleMouseDown(event)}
            onmouseup={(event) => handleMouseUp(event)}
            ondblclick={(event) => {
              cell.isWritable = true;
            }}
          >
            {cell.value}
          </button>
        {/if}
      {/each}
    </div>
  {/each}
</div>
<button onclick={createPlot}>Create Plot</button>
<div style="width: 500px"><canvas id="chart"></canvas></div>

<style>
  .grid {
    display: flex;
    flex-direction: row;
  }
  .row {
    display: flex;
    flex-direction: column;
  }
  .cell {
    width: 6em;
    height: 3em;
    border: 1px solid #ccc;
    box-sizing: border-box;
    cursor: pointer;
    background-color: white;
  }
  .cell.selected {
    background-color: gray;
  }
</style>
