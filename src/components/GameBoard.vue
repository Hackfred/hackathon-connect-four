<template>
  <svg
    :viewBox="`0 0 ${boardWidth} ${boardHeight}`"
    xmlns="http://www.w3.org/2000/svg"
    class="game-board"
    stroke="none"
  >
    <defs>
      <pattern :id="patternId" patternUnits="userSpaceOnUse" :width="cellSize" :height="cellSize">
        <circle :cx="cellSize / 2" :cy="cellSize / 2" :r="checkerRadius" fill="black"></circle>
      </pattern>
      <mask :id="maskId">
        <rect :width="cellSize" :height="boardHeight" fill="white"></rect>
        <rect :width="cellSize" :height="boardHeight" :fill="pattern"></rect>
      </mask>
    </defs>
    <board-column
      v-for="col in cols"
      :key="col"
      :checkers="colCheckers(col)"
      :col="col"
      :color="'cadetblue'"
      :cellSize="cellSize"
      :checkerRadius="checkerRadius"
      :boardHeight="boardHeight"
      :rowCount="rowCount"
      :mask="mask"
      :status="status"
      @drop="drop"
      @land="land"
    ></board-column>
  </svg>
</template>

<script>
import BoardColumn from "./BoardColumn.vue";
import { cssUrl, range } from "../shared/functions.js";

export default {
  name: "GameBoard",
  props: ["checkers", "rowCount", "colCount", "status"],
  components: {
    BoardColumn
  },

  data() {
    return {
      patternId: "cell-pattern",
      maskId: "cell-mask",
      cellSize: 100
    };
  },

  computed: {
    pattern() {
      return cssUrl(this.patternId);
    },
    mask() {
      return cssUrl(this.maskId);
    },

    rows() {
      return range(this.rowCount);
    },
    cols() {
      return range(this.colCount);
    },

    boardWidth() {
      return this.colCount * this.cellSize;
    },
    boardHeight() {
      return this.rowCount * this.cellSize;
    },
    checkerRadius() {
      return this.cellSize * 0.45;
    }
  },

  methods: {
    colCheckers(col) {
      console.log("col", col);
      console.log("checkers", this.checkers);
      return Object.values(this.checkers)
        .filter(c => c.col === col)
        .sort((a, b) => {
          console.log("inside", a, b);
          return a.row - b.row;
        });
    },
    drop(data) {
      console.log("emit drop");
      this.$emit("drop", data);
    },
    land() {
      console.log("emit land");
      this.$emit("land");
    }
  }
};
</script>
