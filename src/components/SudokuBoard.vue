<template>
<div class="container">
    <div class="board">
        <div class="row" v-for="(row, rowIndex) in board" :key="rowIndex">
            <div class="cell" v-for="(cell, cellIndex) in row" :key="cellIndex">
                <input :id="rowIndex + '|' + cellIndex" type="/text" @keypress="validate($event, rowIndex, cellIndex)" maxlength="1" v-model="board[rowIndex][cellIndex]" />
            </div>
        </div>
    </div>
    <div class="buttons">
        <button @click="reset">Reset</button>
        <button @click="startSolve">Solve</button>
    </div>
</div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { cloneDeep } from "lodash";

@Component
export default class SudokuBoard extends Vue {
    data() {
        return {
            board: [
             ["", "", "", "", "", "", "", "", ""],
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],
            ],
            steps: []
        }
    }
    validate(event: KeyboardEvent, row: number, col: number) {
        var charCode = (event.which) ? event.which : event.keyCode;
        
        if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
            event.preventDefault();
            this.$data.board[row][col] = "";
        } else {
            this.$data.board[row][col] = parseInt(this.$data.board[row][col]);
        }
    }
    reset() {
        this.$data.board = [
             ["", "", "", "", "", "", "", "", ""],
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],   
             ["", "", "", "", "", "", "", "", ""],
        ];
    }
    startSolve() {
        const init = cloneDeep(this.$data.board);
 
        const done = this.solve();

        if (done) {
            this.reset();
            console.log(init);
            
            this.$data.board = init;
            this.$data.steps.forEach((state: any, index: number) => {
                setTimeout(() => {
                    this.$data.board[state[0]][state[1]] = state[2];
                    Vue.set(this.$data.board[state[0]], state[1], state[2] );    
                }, index * 10)
            })
            
        }
    }
    solve(): boolean {
        //Checkl if already solved
        if (this.emptyCell()[0] == 9 || this.emptyCell()[1] == 9)
        {
            return true;
        }

        // Get an empty cell
        const cell = this.emptyCell();
        const row = cell[0];
        const col = cell[1];

        // Iterate through possible numbers
        for (let num = 1; num <= 9; num++)
        {
            //Check if current attempt works
            if (this.isSafe(row, col, num))
            {
                //Assign for checking
                this.$data.board[row][col] = num;
                this.$data.steps.push([row, col, num])

                //Search next value in stack
                if (this.solve())
                {
                    return true;
                }
                //If we get to this point, the previous check failed, so we mark as blank and check next value
                this.$data.board[row][col] = "";
                this.$data.steps.push([row, col, ""])
            }
        }
        //Backtrack
        return false;
    }
    isInCol(col: number, value: number): boolean {
        if (col == 9) {
            return false;
        }
        for (let row = 0; row < 9; row++) {
            if (this.$data.board[row][col] == value)
            {
                return true;
            }
        }
        return false;
    }
    isInRow(row: number, value: number): boolean {
        if (row == 9) {
            return false;
        }
        for (let col = 0; col < 9; col++) {
            if (this.$data.board[row][col] == value)
            {
                return true;
            }
        }
        return false;
    }
    isInBox(startRow: number, startColumn: number, value: number): boolean
    {
        if (startRow == 9 || startColumn == 9) {
            return false;
        }
        for (let row = 0; row < 3; row++) {
            for (let col = 0; col < 3; col++) {
                if (this.$data.board[row + startRow][col + startColumn] == value)
                {
                    return true;
                }
        }
        }
        return false;
    }
    isSafe(row: number, col: number, value: number): boolean
    {
        return !this.isInRow(row, value) &&
            !this.isInCol(col, value) &&
            !this.isInBox(row - row % 3, col - col % 3, value);
    }
    emptyCell(): number[] {
        for (let row = 0; row < 9; row++) {
            for (let col = 0; col < 9; col++) {
                if (this.$data.board[row][col] == "")
                {
                    return [row, col];
                }
            } 
        }
        return [9, 9];
    }
}
</script>
<style lang="scss">
.container {
    display: flex;
    flex-direction: column;
    align-content: center;
}

.board {
    margin: auto;
}

.row {
    display: flex;
    flex-direction: row;
    width: auto;
}

.cell {
    // padding: 0 8px 0 8px;
    padding: 8px;
}

.cell > input {
    height: 48px;
    width: 48px;
    text-align: center;
    vertical-align: center;
    font-size: 3rem;
}

.buttons {
    padding: 24px;
}
</style>