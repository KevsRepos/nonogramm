<script lang="ts">
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

let matrix: Array<Field> = $state(buildMatrix(matrixShape));
let fails = $state(0);
let won = $derived(!matrix.filter((val) => val.hidden).length)

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
            numbersTop[col] ||= [1];
            numbersLeft[row] ||= [1];
            
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

<main>
    <game-view>
        <numbers-left style="grid-template-rows: repeat({numRows}, 4rem);">
            {#each numbersLeft as row}
                <div>{row.join(',')}</div>
            {/each}
        </numbers-left>
        
        <numbers-top style="grid-template-columns: repeat({numCols}, 4rem);">
            {#each numbersTop as col}
                <div>{col.join(',')}</div>
            {/each}
        </numbers-top>
        
        <game-matrix style="grid-template-columns: repeat({numCols}, 1fr); grid-template-rows: repeat({numRows}, 1fr);">
            {#each matrix as field, index}
                <game-field type="button" onclick={() => reveal(field, index)}>
                    <pre>{index} </pre>
                    {#if field.hidden}
                        #
                    {/if}
                    <!--{#if field.hidden}-->
                        {field.value}
                    <!--{/if}-->
                </game-field>
            {/each}
        </game-matrix>
    </game-view>

    <game-controls>
        <button onclick={() => btnAction = 0}>0</button>
        <button onclick={() => btnAction = 1}>1</button>

        <hr />
        <div>Fails: {fails}</div>
    </game-controls>
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
main {
    display: flex;
    justify-content: center;
    flex-direction: column;
    margin-top: 2rem;
}
game-view {
    position: relative;
    display: block;
    margin: auto;
}
numbers-top {
    display: grid;
    grid-template-rows: 1;
}
numbers-left {
    position: absolute;
    bottom: 0;
    transform: translateX(-100%);
    display: inline-grid;
    grid-template-columns: 1;
}
numbers-top div {
    display: flex;
    justify-content: center;
}
numbers-left div {
    display: flex;
    align-items: center;
    justify-content: flex-end;
}
game-matrix {
    display: grid;
    width: max-content;
}
game-field {
    border: 1px green solid;
    height: 4rem;
    width: 4rem;
    display: flex;
    justify-content: center;
    align-items: center;
}
game-controls {
    display: block;
    margin: auto;
    margin-top: 2rem;
}
</style>