<script lang="ts">
  type Cell = {
    x: number;
    y: number;
    isSelected: boolean;
    value: string;
  };

  const rows = 10;
  const cols = 10;

  let grid: Cell[][] = $state(
    Array.from({ length: rows }, (_, y) =>
      Array.from({ length: cols }, (_, x) => ({
        x,
        y,
        isSelected: false,
        value: "",
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
      cell.isSelected = true;
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
</script>

<div class="grid">
  {#each grid as row}
    <div class="row">
      {#each row as cell}
        <button
          class="cell {cell.isSelected ? 'selected' : ''}"
          onmousedown={(event) => handleMouseDown(event)}
          onmouseup={(event) => handleMouseUp(event)}
        >
          {cell.value}
        </button>
      {/each}
    </div>
  {/each}
</div>

<style>
  .grid {
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    gap: 2px;
  }
  .row {
    display: contents;
  }
  .cell {
    padding: 10px;
    border: 1px solid #ccc;
    text-align: center;
    cursor: pointer;
    user-select: none;
    background-color: white;
  }
  .cell.selected {
    background-color: #007bff;
    color: white;
  }
</style>
