<script lang="ts">
import { IconPointFilled, IconX } from "@tabler/icons-svelte";

let btnAction: 1|0 = $state(1);

interface Field {
    value: 1|0,
    hidden: boolean
}

const matrixShape: Array<1|0> = [
    0, 0, 1, 1, 1, 1,
    0, 1, 1, 0, 0, 1,
    0, 0, 1, 0, 0, 1,
    0, 1, 1, 0, 0, 1,
    1, 0, 0, 1, 0, 0,
    1, 0, 1, 0, 1, 0,
];

const buildMatrix = (matrixShape: Array<1|0>): Array<Field> => {
    return matrixShape.map((val) => ({value: val, hidden: true}))
}

let seconds = $state(0);
let minutes = $derived(Math.floor(seconds / 60));
let time = $derived(`${minutes}:${seconds}`);

$effect(() => {
    const interval = setInterval(() => {
        seconds++;
    }, 1000);

    return () => {
        clearInterval(interval);
    }
});

let matrix: Array<Field> = $state(buildMatrix(matrixShape));
let fails = $state(0);
let won = $derived(!matrix.filter((val) => val.hidden).length);

const rowLength = 6;
const numCols = rowLength;
const numRows = matrixShape.length / rowLength;

const createNumbers = () => {
    const numbersTop: Array<Array<number>> = [];
    const numbersLeft: Array<Array<number>> = [];
    
    for(let i = 0; i < matrix.length; i++) {
        const row = Math.floor(i / numCols);
        const col = i - rowLength * row;
        
        if(matrix[i].value === 1) {
            numbersTop[col] ||= [];
            numbersLeft[row] ||= [];
            
            if(matrix[i - rowLength]?.value >= 1) {
                numbersTop[col].splice(numbersTop[col].length - 1, 1, numbersTop[col].at(-1)! + 1);
            } else {
                numbersTop[col].push(1);
            }

            if(col > 0 && matrix[i - 1]?.value >= 1) {
                numbersLeft[row].splice(numbersLeft[row].length - 1, 1, numbersLeft[row].at(-1)! + 1);
            } else {
                numbersLeft[row].push(1);
            }
        }
    }

    return [numbersLeft, numbersTop];
}

const reveal = (field: Field, index: number) => {
    field.hidden = false;

    if(btnAction !== field.value) {
        fails++;
    }

    const row = Math.floor(index / numCols);
    const col = index - rowLength * row;

    const currentRow = matrix.slice(index - col, index - col + rowLength);
    checkAndReveal(currentRow);

    const currentCol = [];
    for (let i = 0; i < numCols; i++) {
        currentCol.push(matrix[i * rowLength + col]);
    }
    checkAndReveal(currentCol);
}

const checkAndReveal = (fields: Array<Field>) => {
    const allRevealed = fields.every((val) => !(val.hidden && val.value === 1));
    
    if(allRevealed) {
        for(const field of fields) {
            field.hidden = false;
        }
    }
}

const reset = () => {
    matrix = buildMatrix(matrixShape);
    fails = 0;
}

const [numbersLeft, numbersTop] = createNumbers();
</script>

{#if fails >= 3}
    <game-over>
        <h1>Game Over!</h1>
        <button onclick={reset}>Retry</button>
    </game-over>
{/if}

{#if won}
    <game-over>
        <h1>You won!</h1>
        <button onclick={reset}>Again</button>
    </game-over>
{/if}

<main class="flex justify-center flex-col mt-8">
    <game-view class="relative block m-auto">
        <numbers-left class="absolute bottom-0 -translate-x-full pr-1 grid grid-cols-1" style="grid-template-rows: repeat({numRows}, 3rem);">
            {#each numbersLeft as row}
                <div class="flex items-center justify-end">{row.join(',')}</div>
            {/each}
        </numbers-left>
        
        <numbers-top class="grid grid-rows-1 mx-auto" style="grid-template-columns: repeat({numCols}, 3rem);">
            {#each numbersTop as col}
                <div class="flex justify-center">{col.join(',')}</div>
            {/each}
        </numbers-top>
        
        <game-matrix class="grid h-max mx-auto" style="grid-template-columns: repeat({numCols}, 3rem); grid-template-rows: repeat({numRows}, 3rem);">
            {#each matrix as field, index}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <game-field role="button" tabindex="0" class="gap-px border border-secondary-300 flex justify-center items-center" type="button" onclick={() => reveal(field, index)}>
                    {#if !field.hidden}
                        {#if field.value === 1}
                            <IconPointFilled class="text-green-700" />
                        {:else}
                            <IconX class="text-red-700" />
                        {/if}
                    {/if}
                </game-field>
            {/each}
        </game-matrix>
    </game-view>

    <game-controls class="card w-11/12 mx-auto my-8 p-4 flex justify-center gap-1">
        <button class="btn {btnAction ? 'variant-soft' : 'variant-soft-success'} btn-icon btn-se" onclick={() => btnAction = 0}>
            <IconX class="text-red-700" />
        </button>
        <button class="btn {btnAction ? 'variant-soft-success' : 'variant-soft'} btn-icon" onclick={() => btnAction = 1}>
            <IconPointFilled class="text-green-700" />
        </button>
    </game-controls>

    <div class="flex justify-center">
        Fails:&#10240
        <span class="{fails >= 1 ? 'text-warning-700' : 'text-primary-700'}">{fails}</span>
    </div>

    <hr class="!border-dashed my-3" />

    <div class="flex justify-center">
        {time}
    </div>
</main>

<style>
* {
    box-sizing: border-box;
}
:global(body) {
    margin: 0;
}
game-over {
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    background-color: rgb(0, 0, 0, .3);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: fixed;
    z-index: 9999;
}
</style>