<template>
  <div class="game-2048">
    <div class="game-header">
      <div class="score">Score: {{ score }}</div>
      <button @click="newGame" class="new-game-btn">New Game</button>
    </div>
    <div class="game-board" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="game-row">
        <div v-for="(cell, colIndex) in row" :key="colIndex" class="game-cell" :class="'cell-' + cell">
          {{ cell || '' }}
        </div>
      </div>
    </div>
    <div class="direction-pad">
      <div class="d-pad-row">
        <button class="d-pad-btn" @click="move('up')">↑</button>
      </div>
      <div class="d-pad-row">
        <button class="d-pad-btn" @click="move('left')">←</button>
        <div class="d-pad-center"></div>
        <button class="d-pad-btn" @click="move('right')">→</button>
      </div>
      <div class="d-pad-row">
        <button class="d-pad-btn" @click="move('down')">↓</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Game2048',
  data() {
    return {
      board: [],
      score: 0,
      touchStartX: 0,
      touchStartY: 0
    }
  },
  mounted() {
    this.newGame()
    window.addEventListener('keydown', this.handleKeydown)
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown)
  },
  methods: {
    newGame() {
      this.board = Array(4).fill().map(() => Array(4).fill(0))
      this.score = 0
      this.addNewTile()
      this.addNewTile()
    },
    addNewTile() {
      const emptyCells = []
      for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 4; j++) {
          if (this.board[i][j] === 0) {
            emptyCells.push({ x: i, y: j })
          }
        }
      }
      if (emptyCells.length > 0) {
        const { x, y } = emptyCells[Math.floor(Math.random() * emptyCells.length)]
        this.board[x][y] = Math.random() < 0.9 ? 2 : 4
      }
    },
    move(direction) {
      const oldBoard = JSON.stringify(this.board)
      let moved = false

      // Helper function to rotate matrix clockwise
      const rotateClockwise = (matrix) => {
        const N = matrix.length;
        const rotated = Array(N).fill().map(() => Array(N).fill(0));
        for (let i = 0; i < N; i++) {
          for (let j = 0; j < N; j++) {
            rotated[j][N-1-i] = matrix[i][j];
          }
        }
        return rotated;
      }

      // Helper function to rotate matrix counterclockwise
      const rotateCounterClockwise = (matrix) => {
        const N = matrix.length;
        const rotated = Array(N).fill().map(() => Array(N).fill(0));
        for (let i = 0; i < N; i++) {
          for (let j = 0; j < N; j++) {
            rotated[N-1-j][i] = matrix[i][j];
          }
        }
        return rotated;
      }

      let newBoard = this.board.map(row => [...row]);

      if (direction === 'left') {
        newBoard = newBoard.map(row => this.mergeLine(row));
      } else if (direction === 'right') {
        newBoard = newBoard.map(row => this.mergeLine([...row].reverse()).reverse());
      } else if (direction === 'up') {
        // Rotate, merge as if left, then rotate back
        newBoard = rotateCounterClockwise(newBoard);
        newBoard = newBoard.map(row => this.mergeLine(row));
        newBoard = rotateClockwise(newBoard);
      } else if (direction === 'down') {
        // Rotate, merge as if right, then rotate back
        newBoard = rotateCounterClockwise(newBoard);
        newBoard = newBoard.map(row => this.mergeLine([...row].reverse()).reverse());
        newBoard = rotateClockwise(newBoard);
      }

      if (JSON.stringify(newBoard) !== oldBoard) {
        this.board = newBoard;
        this.addNewTile();
        moved = true;
      }

      if (!this.canMove()) {
        setTimeout(() => {
          alert('Game Over! Score: ' + this.score);
        }, 300);
      }

      return moved;
    },
    mergeLine(line) {
      let result = line.filter(cell => cell !== 0)
      for (let i = 0; i < result.length - 1; i++) {
        if (result[i] === result[i + 1]) {
          result[i] *= 2
          this.score += result[i]
          result.splice(i + 1, 1)
        }
      }
      while (result.length < 4) {
        result.push(0)
      }
      return result
    },
    canMove() {
      for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 4; j++) {
          if (this.board[i][j] === 0) return true
          if (i < 3 && this.board[i][j] === this.board[i + 1][j]) return true
          if (j < 3 && this.board[i][j] === this.board[i][j + 1]) return true
        }
      }
      return false
    },
    handleKeydown(event) {
      switch(event.key) {
        case 'ArrowUp':
          event.preventDefault()
          this.move('up')
          break
        case 'ArrowDown':
          event.preventDefault()
          this.move('down')
          break
        case 'ArrowLeft':
          event.preventDefault()
          this.move('left')
          break
        case 'ArrowRight':
          event.preventDefault()
          this.move('right')
          break
      }
    },
    touchStart(event) {
      this.touchStartX = event.touches[0].clientX
      this.touchStartY = event.touches[0].clientY
    },
    touchMove(event) {
      event.preventDefault()
    },
    touchEnd(event) {
      const touchEndX = event.changedTouches[0].clientX
      const touchEndY = event.changedTouches[0].clientY
      const dx = touchEndX - this.touchStartX
      const dy = touchEndY - this.touchStartY
      
      if (Math.abs(dx) > Math.abs(dy)) {
        if (dx > 0) {
          this.move('right')
        } else {
          this.move('left')
        }
      } else {
        if (dy > 0) {
          this.move('down')
        } else {
          this.move('up')
        }
      }
    }
  }
}
</script>

<style scoped>
.game-2048 {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.score {
  font-size: 24px;
  font-weight: bold;
}

.new-game-btn {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #8f7a66;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.game-board {
  background-color: #bbada0;
  padding: 10px;
  border-radius: 6px;
  touch-action: none;
}

.game-row {
  display: flex;
  margin-bottom: 10px;
}

.game-row:last-child {
  margin-bottom: 0;
}

.game-cell {
  width: 80px;
  height: 80px;
  margin-right: 10px;
  background-color: rgba(238, 228, 218, 0.35);
  border-radius: 3px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
  font-weight: bold;
  color: #776e65;
}

.game-cell:last-child {
  margin-right: 0;
}

.direction-pad {
  margin-top: 30px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.d-pad-row {
  display: flex;
  gap: 5px;
  align-items: center;
}

.d-pad-btn {
  width: 60px;
  height: 60px;
  background-color: #8f7a66;
  border: none;
  border-radius: 10px;
  color: white;
  font-size: 24px;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: background-color 0.2s;
}

.d-pad-btn:hover {
  background-color: #9f8a76;
}

.d-pad-btn:active {
  background-color: #7f6a56;
}

.d-pad-center {
  width: 60px;
  height: 60px;
}

.cell-2 { background-color: #eee4da; }
.cell-4 { background-color: #ede0c8; }
.cell-8 { background-color: #f2b179; color: #f9f6f2; }
.cell-16 { background-color: #f59563; color: #f9f6f2; }
.cell-32 { background-color: #f67c5f; color: #f9f6f2; }
.cell-64 { background-color: #f65e3b; color: #f9f6f2; }
.cell-128 { background-color: #edcf72; color: #f9f6f2; font-size: 20px; }
.cell-256 { background-color: #edcc61; color: #f9f6f2; font-size: 20px; }
.cell-512 { background-color: #edc850; color: #f9f6f2; font-size: 20px; }
.cell-1024 { background-color: #edc53f; color: #f9f6f2; font-size: 18px; }
.cell-2048 { background-color: #edc22e; color: #f9f6f2; font-size: 18px; }

@media (max-width: 500px) {
  .game-cell {
    width: 60px;
    height: 60px;
    font-size: 20px;
  }
  
  .cell-128, .cell-256, .cell-512 { font-size: 16px; }
  .cell-1024, .cell-2048 { font-size: 14px; }
}
</style>
