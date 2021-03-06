<script lang="ts" setup>
import { reactive, onMounted, onBeforeUnmount } from "vue";
import { Tetromino, TETROMINO_TYPE } from '../common/Tetromino'
import { Field } from '../common/Field'

import TetrominoPreviewComponent from '../components/TetrominoPreviewComponent.vue'

let staticField = new Field();
const tetris = reactive({
  field: new Field(),
  score: 0
});
const tetromino = reactive({
  current: Tetromino.random(),
  position: { x: 3, y: 0 },
  rotate: 0,
  next: Tetromino.random()
});

const currentTetrominoData = () => {
  return Tetromino.rotate(tetromino.rotate, tetromino.current.data)
}

const classBlockColor = (_x: number, _y: number): string => {
  const type = tetris.field.data[_y][_x];
  if(type > 0) {
    return Tetromino.id(type as TETROMINO_TYPE);
  }

  const { x, y } = tetromino.position;

  const data = currentTetrominoData()

  if (y <= _y && _y < y + data.length) {
    const cols = data[_y - y];
    if (x <= _x && _x < x + cols.length) {
      if (cols[_x - x] > 0) {
        return Tetromino.id(cols[_x - x] as TETROMINO_TYPE);
      }
    }
  }

  return "";
}

const canDropCurrentTetromino = (): boolean => {
  const { x, y } = tetromino.position;
  const droppedPosition = {x, y: y + 1};

  const data = currentTetrominoData()
  return tetris.field.canMove(data, droppedPosition);
}

const nextTetrisField = () => {
  const data = currentTetrominoData()
  const position = tetromino.position;

  tetris.field.update(data, position);
  const { field, score } = deleteLine()

  staticField = new Field(field)
  tetris.field = Field.deepCopy(staticField);
  tetris.score += score

  tetromino.current = tetromino.next;
  tetromino.next = Tetromino.random();
  tetromino.rotate = 0;
  tetromino.position = { x: 3, y: 0 };
}

const onKeyDown = (e: KeyboardEvent) => {
  switch (e.key) {
    case " ": {
      const nextRotate = (tetromino.rotate + 1) % 4;
      const data = Tetromino.rotate(nextRotate, tetromino.current.data)
      if (tetris.field.canMove(data, tetromino.position)) {
        tetromino.rotate = nextRotate
      }
    }
      break;
    case "Down":
    case "ArrowDown":
      if(canDropCurrentTetromino()) {
        tetromino.position.y++;
        resetDrop();
      } else {
        nextTetrisField();
      }
      break;
    case "Up":
    case "ArrowUp":
      while(canDropCurrentTetromino()) {
        tetromino.position.y++;
        resetDrop();
      }
      nextTetrisField()
      break;
    case "Left":
    case "ArrowLeft": {
      const data = currentTetrominoData()
      const { x, y } = tetromino.position
      const leftPosition = {x: x - 1, y};
      if(tetris.field.canMove(data, leftPosition)) {
        tetromino.position.x--;
      }
    }
      break;
    case "Right":
    case "ArrowRight": {
      const data = currentTetrominoData()
      const { x, y } = tetromino.position
      const rightPosition = {x: x + 1, y};
      if(tetris.field.canMove(data, rightPosition)) {
        tetromino.position.x++;
      }
    }
      break;
  }
}

onMounted(function() {
  document.addEventListener('keydown', onKeyDown)
})

const deleteLine = () => {
  let score = 0;
  const field = tetris.field.data.filter((row) => {
    if (row.every(col => col > 0)) {
      score++;
      return false;
    }
    return true;
  });

  for (let i = 0; i < score; i++) {
    field.unshift(new Array(field[0].length).fill(0));
  }

  return { score, field };
};

onBeforeUnmount(function() {
  document.removeEventListener('keydown', onKeyDown)
})

const resetDropInterval = () => {
  let intervalId = -1;

  return () => {
    if (intervalId !== -1) clearInterval(intervalId);

    intervalId = setInterval(() => {
      tetris.field = Field.deepCopy(staticField);

      if(canDropCurrentTetromino()) {
        tetromino.position.y++;
      } else {
        nextTetrisField();
      }
    }, 1 * 1000);
  };
};

const resetDrop = resetDropInterval();
resetDrop();

</script>

<template>
  <h1>プレイ画面</h1>
  <h2>ユーザ名: {{ $route.query.name }}</h2>
  <div class="container">
    <div class="tetris">
      <table class="field" style="border-collapse: collapse">
        <tr v-for="(row, y) in tetris.field.data" v-bind:key="row+y">
          <td
            class="block"
            v-for="(col, x) in row"
            v-bind:class="classBlockColor(x, y)"
            v-bind:key="col+x"
          />
        </tr>
      </table>
    </div>
    <div class="information">
      <TetrominoPreviewComponent v-bind:tetromino="tetromino.next.data"/>
      <ul class="data">
        <li>スコア: {{ tetris.score }}</li>
      </ul>
    </div>
  </div>
</template>

<!-- スタイルシートに SCSS 記法 (Sass) を利用する -->
<style lang="scss" scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: stretch;
}

.field {
  border: ridge 0.4em #2c3e50;
  border-top: none;
}

.block {
  width: 1em;
  height: 1em;
  border: 0.1px solid #95a5a6;

  /*
    各テトリミノに対応した色を扱うクラス定義
    .block-i, .block-o のようにクラスが定義される
  */
  &-i {
    background: #3498db;
  }
  &-o {
    background: #f1c40f;
  }
  &-t {
    background: #9b59b6;
  }
  &-j {
    background: #1e3799;
  }
  &-l {
    background: #e67e22;
  }
  &-s {
    background: #2ecc71;
  }
  &-z {
    background: #e74c3c;
  }
}

/** テトリスに関する情報をテトリスのフィールドの右に表示する **/
.information {
  position: relative;
  margin-left: 0.5em;

  ul.data {
    list-style: none;
    position: absolute;
    font-size: 1.3em;
    padding-left: 0;
    bottom: 0;
  }
}
</style>