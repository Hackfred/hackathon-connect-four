<template>
  <div id="app">
    <h2>Vier gewinnt</h2>
    <GameContainer></GameContainer>
  </div>
</template>

<script>
import GameContainer from "./components/GameContainer";
import GameBoard from "./components/GameBoard";
import BoardColumn from "./components/BoardColumn";
import BoardChecker from "./components/BoardChecker";

export default {
  name: "App",
  components: {
    GameContainer: GameContainer,
    GameBoard,
    BoardChecker,
    BoardColumn
  }
};
</script>

<style>
body {
  background-color: azure;
}

a {
  text-decoration: none;
}

.logo h3 {
  margin: 0;
}

.game-board {
  border: 5px cadetblue solid;

  width: 350px;
  margin: 0 auto;
}

.column {
  cursor: pointer;
}

.column rect.play:hover {
  fill-opacity: 0.75;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
