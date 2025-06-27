<script lang="ts">
    import type { Cell } from "$lib/types";
    import { onMount } from "svelte";
    let { cell }: { cell: Cell } = $props();
    onMount(() => {
        if (cell.isWritable) {
            const input = document.querySelector("input");
            if (input) {
                input.focus();
            }
        }
    });
</script>

{#if cell.isWritable}
    <input
        type="text"
        bind:value={cell.value}
        class="cell"
        onkeydown={(e) => {
            if (e.key === "Enter") {
                e.preventDefault();
                cell.isWritable = false;
            }
        }}
        onblur={() => (cell.isWritable = false)}
    />
{:else}
    <button
        class="cell"
        class:selected={cell.isSelected}
        onclick={() => (cell.isWritable = true)}
    >
        {cell.value}
    </button>
{/if}
