<template>
  <div>
    <p>{{ instructions }}</p>
    <game-board
      :checkers="checkers"
      :rowCount="rowCount"
      :colCount="colCount"
      :status="status"
      @drop="drop"
      @land="land"
    ></game-board>
    <p v-if="gameOver">
      {{ overMessage }}
      <a href="#" @click="reset">Play again</a>
    </p>
    <p v-else>{{ whoseTurn }}</p>
  </div>
</template>

<script>
import Vue from "vue";
import GameBoard from "./GameBoard.vue";
import { PLAY, RED, OVER, EMPTY, BLACK } from "../shared/constants.js";
import { titleize, min, max, key } from "../shared/functions.js";

export default {
  name: "GameContainer",

  components: {
    GameBoard
  },

  data() {
    return {
      checkers: {},
      isLocked: false,
      playerColor: RED,
      rowCount: 6,
      colCount: 7,
      status: PLAY,
      instructions: "Klicke auf eine der Spalten, um einen Stein zu setzen.",
      winner: undefined
    };
  },

  computed: {
    overMessage() {
      if (this.winner) {
        return `${titleize(this.winner.color)} wins!`;
      } else {
        return `It's a draw!`;
      }
    },

    moves() {
      return Object.values(this.checkers);
    },

    whoseTurn() {
      return `${titleize(this.playerColor)} goes ${
        this.moves.length === 0 ? "first" : "next"
      }`;
    },

    gameOver() {
      return this.status === OVER;
    }
  },

  methods: {
    key,

    reset() {
      this.winner = undefined;
      this.isLocked = false;
      this.status = PLAY;
      this.checkers = {};
    },

    toggleColor() {
      if (this.playerColor === RED) {
        this.playerColor = BLACK;
      } else {
        this.playerColor = RED;
      }
    },

    setChecker({ row, col }, attrs = {}) {
      const checker = this.getChecker({ row, col });
      return Vue.set(this.checkers, key(row, col), { ...checker, ...attrs });
    },

    getChecker({ row, col }) {
      return this.checkers[key(row, col)] || { row, col, color: "empty" };
    },

    drop({ col, row }) {
      if (this.isLocked) return;

      this.isLocked = true;
      const color = this.playerColor;

      console.log("setting checker", key(row, col), { row, col, color });
      this.setChecker({ row, col }, { color });

      this.checkForDraw() || this.checkForWinFrom({ row, col });
      this.toggleColor();
    },

    land() {
      if (this.isDraw) return this.displayDraw();

      if (this.winner) {
        this.displayWin(this.winner);
      } else {
        this.isLocked = false;
      }
    },

    checkForDraw() {
      this.isDraw =
        Object.keys(this.checkers).length === this.rowCount * this.colCount;
    },

    getWinner(...segment) {
      if (segment.length !== 4) return false;
      const checkers = segment.map(([row, col]) =>
        this.getChecker({ row, col })
      );
      const color = checkers[0].color;
      if (color === EMPTY) return false;
      if (checkers.every(c => c.color === color)) return { color, checkers };
      return false;
    },

    checkHorizontalSegments({ focalRow, minCol, maxCol }) {
      for (let row = focalRow, col = minCol; col <= maxCol; col++) {
        const winner = this.getWinner(
          [row, col],
          [row, col + 1],
          [row, col + 2],
          [row, col + 3]
        );
        if (winner) return winner;
      }
    },

    checkVerticalSegments({ focalRow, focalCol, minRow, maxRow }) {
      for (let col = focalCol, row = minRow; row <= focalRow; row++) {
        const winner = this.getWinner(
          [row, col],
          [row + 1, col],
          [row + 2, col],
          [row + 3, col]
        );
        if (winner) return winner;
      }
    },

    checkForwardSlashSegments({
      focalRow,
      focalCol,
      minRow,
      minCol,
      maxRow,
      maxCol
    }) {
      const startForwardSlash = (row, col) => {
        console.log(
          "startForwardSlash",
          focalRow,
          focalCol,
          minRow,
          minCol,
          maxRow,
          maxCol,
          row,
          col
        );
        while (row > minRow && col > minCol) {
          row--;
          col--;
        }
        return [row, col];
      };

      // little test
      let row, col;
      for (
        [row, col] = startForwardSlash(focalRow, focalCol);
        row <= maxRow && col <= maxCol;
        row++, col++
      ) {
        const winner = this.getWinner(
          [row, col],
          [row + 1, col + 1],
          [row + 2, col + 2],
          [row + 3, col + 3]
        );
        if (winner) return winner;
      }
    },

    checkBackwardSlashSegments({
      focalRow,
      focalCol,
      minRow,
      minCol,
      maxRow,
      maxCol
    }) {
      const startBackwardSlash = (row, col) => {
        while (row < maxRow && col > minCol) {
          row++;
          col--;
        }
        return [row, col];
      };
      let row, col;
      for (
        [row, col] = startBackwardSlash(focalRow, focalCol);
        row >= minRow && col <= maxCol;
        row--, col++
      ) {
        const winner = this.getWinner(
          [row, col],
          [row - 1, col + 1],
          [row - 2, col + 2],
          [row - 3, col + 3]
        );
        if (winner) return winner;
      }
    },

    checkForWinFrom(lastChecker) {
      if (!lastChecker) return;
      const { row: focalRow, col: focalCol } = lastChecker;

      const minCol = min(focalCol);
      const maxCol = max(focalCol, this.colCount - 1);
      const minRow = min(focalRow);
      const maxRow = max(focalRow, this.rowCount - 1);
      const coords = { focalRow, focalCol, minRow, minCol, maxRow, maxCol };

      this.winner =
        this.checkHorizontalSegments(coords) ||
        this.checkVerticalSegments(coords) ||
        this.checkForwardSlashSegments(coords) ||
        this.checkBackwardSlashSegments(coords);
    },

    displayDraw() {
      this.status = OVER;
    },

    displayWin(winner) {
      this.winner = winner;
      this.status = OVER;
      this.winner.checkers.forEach(checker => {
        this.setChecker(checker, { isWinner: true });
      });
      console.log("Win!", winner);
    }
  }
};
</script>
