<script lang="ts" setup>
import { reactive } from "vue";
import { Tetromino, TETROMINO_TYPE } from '../common/Tetromino'
import { Field } from '../common/Field'

let staticField = new Field();
const tetris = reactive({ field: new Field() })
const tetromino = reactive({
  current: Tetromino.random(),
  position: { x: 3, y: 0 }
})

const row = 20;
const column = 10;

const field = new Array(row);

for (let i = 0; i < row; i++) {
  const fieldColumn = new Array(column).fill(0);
  field[i] = fieldColumn;
}

// 画面最上部の左端に縦の I-テトリミノを配置する
field[0][0] = 1
field[1][0] = 1
field[2][0] = 1
field[3][0] = 1

const classBlockColor = (_x: number, _y: number): string => {
  const type = tetris.field.data[_y][_x]
  if(type > 0) {
    return Tetromino.id(type as TETROMINO_TYPE)
  }

  const { x, y } = tetromino.position
  const { data } = tetromino.current

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
  const { x, y } = tetromino.position
  const droppedPosition = {x, y: y + 1};

  const data = tetromino.current.data
  return tetris.field.canMove(data, droppedPosition)
}

const nextTetrisField = () => {
  const data = tetromino.current.data
  const position = tetromino.position

  tetris.field.update(data, position);

  staticField = new Field(tetris.field.data)
  tetris.field = Field.deepCopy(staticField)

  tetromino.current = Tetromino.random()
  tetromino.position = { x: 3, y: 0 }
}

setInterval(() => {
  tetris.field = Field.deepCopy(staticField)

  if(canDropCurrentTetromino()) {
      tetromino.position.y++
  } else {
    nextTetrisField()
  }
}, 1 * 100)
tetris.field.update(tetromino.current.data, tetromino.position);

</script>

<template>
  <h1>プレイ画面</h1>
  <h2>ユーザ名: {{ $route.query.name }}</h2>
  <div class="container">
    <table class="field" style="border-collapse: collapse">
      <tr v-for="(row, y) in field" :key="row+y">
       <!-- テトリスのフィールドの各マス目にその状態を描画する (0: 空白, 1: I-テトリミノ, etc.) -->
        <td
          :key="col+x"
          class="block"
          v-for="(col, x) in row"
          v-bind:class="classBlockColor(x, y)"
        />
      </tr>
    </table>

  </div>
</template>

<!-- スタイルシートに SCSS 記法 (Sass) を利用する -->
<style lang="scss" scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
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

</style>