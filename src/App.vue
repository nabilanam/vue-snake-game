<template>
  <div class="game">
    <div class="board">
      <div class="row" v-for="(_, row) in boardSize" :key="'r' + row">
        <div class="cell" v-for="(_, col) in boardSize" :key="'c' + col">
          <template v-if="hasFood(col, row) && food.isBig"> O </template>
          <template v-else-if="hasFood(col, row)"> o </template>

          <template v-if="hasSnakeBody(col, row)"> X </template>
        </div>
      </div>
    </div>

    <div class="score">Score: {{ score }}</div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      boardSize: 12,
      dir: {
        x: 1,
        y: 0,
      },
      backDir: {
        x: -1,
        y: 0,
      },
      snake: [
        {
          x: 2,
          y: 0,
        },
        {
          x: 1,
          y: 0,
        },
        {
          x: 0,
          y: 0,
        },
      ],
      speed: 4.2,
      gameInterval: null,
      bigFoodTimeout: null,
      bigFoodMaxTime: 5000,
      food: {
        x: -1,
        y: -1,
        isBig: false,
        spawnTime: 0,
      },
      foodStreak: 0,
      shouldSpawnFood: true,
      score: 0,
    };
  },
  methods: {
    hasFood(x, y) {
      return this.food.x == x && this.food.y == y;
    },
    hasSnakeBody(x, y) {
      return (
        this.snake.find(
          (item) => Math.floor(item.x) == x && Math.floor(item.y) == y
        ) != undefined
      );
    },
    setupInput() {
      window.addEventListener(
        "keydown",
        function (e) {
          if (e.keyCode == 37) {
            this.dir.x = -1;
            this.dir.y = 0;
          } else if (e.keyCode == 38) {
            this.dir.x = 0;
            this.dir.y = -1;
          } else if (e.keyCode == 39) {
            this.dir.x = 1;
            this.dir.y = 0;
          } else if (e.keyCode == 40) {
            this.dir.x = 0;
            this.dir.y = 1;
          }

          if (this.dir.x == this.backDir.x && this.dir.y == this.backDir.y) {
            this.dir.x *= -1;
            this.dir.y *= -1;
          } else {
            this.backDir.x = this.dir.x * -1;
            this.backDir.y = this.dir.y * -1;
          }
        }.bind(this)
      );
    },
    gameLoop() {
      if (this.shouldSpawnFood) this.spawnFood();

      let head = this.snake[0];
      this.snake.unshift({ x: head.x + this.dir.x, y: head.y + this.dir.y });
      if (this.foodStreak > 0 && this.foodStreak % 5 == 0) {
        this.foodStreak = 0;
      } else {
        this.snake.splice(this.snake.length - 1, 1);
      }

      head = this.snake[0];
      this.handleRoomEdge(head);
      if (this.hasSelfCollided(head)) clearInterval(this.gameInterval);
      if (this.hasFood(head.x, head.y)) this.eatFood();
    },
    spawnFood() {
      let x, y;

      do {
        x = Math.floor(Math.random() * this.boardSize);
        y = Math.floor(Math.random() * this.boardSize);
      } while (this.hasSnakeBody(x, y));

      this.food.x = x;
      this.food.y = y;
      this.food.isBig = this.foodStreak > 0 && this.foodStreak % 5 == 0;
      this.food.spawnTime = new Date().getTime();
      this.shouldSpawnFood = false;

      if (this.food.isBig) {
        this.bigFoodTimeout = setTimeout(() => {
          this.food.isBig = false;
          this.foodStreak = 0;
          clearTimeout(this.bigFoodTimeout);
        }, this.bigFoodMaxTime);
      }
    },
    eatFood() {
      this.food.x = -1;
      this.score += this.food.isBig
        ? Math.floor(
            (this.bigFoodMaxTime - (new Date().getTime() - this.food.spawnTime)) / 100
          )
        : 1;
      this.shouldSpawnFood = true;
      this.foodStreak++;
    },
    hasSelfCollided(head) {
      return (
        this.snake.filter((part) => part.x == head.x && part.y == head.y)
          .length > 1
      );
    },
    handleRoomEdge(head) {
      if (head.x >= this.boardSize) {
        head.x = 0;
      } else if (head.x < 0) {
        head.x = this.boardSize - 1;
      }
      if (head.y >= this.boardSize) {
        head.y = 0;
      } else if (head.y < 0) {
        head.y = this.boardSize - 1;
      }
    },
    init() {
      this.setupInput();

      this.gameInterval = setInterval(this.gameLoop, 1000 / this.speed);
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style>
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.game {
  display: flex;
}

.board {
  display: flex;
  flex-direction: column;
}

.score {
  margin: 10px;
  font-size: 18px;
  font-weight: bold;
  color: brown;
}

.row {
  display: flex;
}

.cell {
  height: 40px;
  width: 40px;
  font-size: 20px;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ccc;
  box-sizing: border-box;
}
</style>