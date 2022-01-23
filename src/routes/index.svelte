<script lang="ts">
import { onMount } from 'svelte';

    import { blur, crossfade, draw, fade, fly, scale, slide } from 'svelte/transition'
    
    let ready = false
    let word = ""
    onMount(() => {
        fetch("./words.json").then(
            res => {
                res.json().then(
                    j => {
                        word = j.words[Math.floor(Math.random()*j.words.length)].toUpperCase()
                        ready = true
                    }
                )
            }
        )
    })

    let currentGuess = ""
    let currentRow = 0


    enum CellValues {
        WRONG,
        CORRECT,
        WRONG_PLACE
    }
    let cellValues = Array(6).fill(0).map(
        () => new Array(5).fill(0).map(() => CellValues.WRONG)
    )
    console.log(cellValues)

    const Range = (a: number, b: number = null) => {
        if (b == null) {
            b = a
            a = 0
        }
        let arr = new Array(b-a).fill(0)
        return arr.map((_, i) => i+a)
    }
    const onType = (e: KeyboardEvent) => {
        if(currentRow >= 6) return
        if (e.key == "Backspace") {
            if (currentGuess.length > 0) {
                currentGuess = currentGuess.slice(0, currentGuess.length - 1)
                console.log(currentGuess)
                document.getElementById(`${currentRow}:${currentGuess.length}`).innerText = ""
            }
            return
        }
        if (e.key == "Enter") {
            if (currentGuess.length < 5) {
                alert("Not enough letters")
                return
            }
            for (let i = 0; i < 5; i++) {
                if (currentGuess.charAt(i) == word.charAt(i)) {
                    cellValues[currentRow][i] = CellValues.CORRECT
                } else if (word.indexOf(currentGuess.charAt(i)) != -1) {
                    if(
                        currentGuess.slice(0, i).split("").filter(v => v == currentGuess.charAt(i)).length 
                        <
                        word.split("").filter(v => v == currentGuess.charAt(i)).length
                    )
                        cellValues[currentRow][i] = CellValues.WRONG_PLACE
                }
            }
            currentRow++
            currentGuess = ""
            return
        }
        if (!/^[a-z]/.test(e.key) || currentGuess.length == 5) return
        currentGuess += e.key.toUpperCase()
        document.getElementById(`${currentRow}:${currentGuess.length-1}`).innerText = currentGuess.charAt(currentGuess.length - 1)
    }

    const clearBoard = () => {
        currentGuess = ""
        currentRow = 0
        cellValues = Array(6).fill(0).map(
            () => new Array(5).fill(0).map(() => CellValues.WRONG)
        )
        for(let row = 0; row < 6; row++) {
            for(let col = 0; col < 5; col++) {
                document.getElementById(`${row}:${col}`).innerText = ""
            }
        }
    }

</script>

<svelte:window on:keydown={onType}></svelte:window>

<h1>Wordle: MSPaint edition</h1>
<div id="game">
{#if ready}
    <h2>The word is {word}</h2>
    <div id="grid">
    {#each Range(6) as row}
        {#each Range(5) as col}
            <div 
                class="cell {cellValues[row][col] === CellValues.CORRECT ? 'correct' : cellValues[row][col] === CellValues.WRONG_PLACE ? 'wrong_place' : 'wrong'}" 
                transition:fade="{{ delay: 100*(row+col), duration: 500}}" 
                id="{row}:{col}"
            ></div>
        {/each}
    {/each}
    </div>
    <button on:click={clearBoard}>Clear board</button>
{/if}
</div>

<style>
    #game {
        margin: 0 auto;
    }
    #grid {
        margin: 0 auto;
        display: grid;
        grid-template-columns: repeat(5, 5%);
        gap: 1% 0.5%;
        padding-bottom: 20px;
    }
    .cell {
        transition: background-color 250ms ease-in; 
        color: white;
        aspect-ratio: 1;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .wrong {
        background-color: #222;
    }
    .correct {
        background-color: rgb(14, 177, 14)
    }
    .wrong_place {
        background-color: rgb(246, 223, 13)
    }
</style>

