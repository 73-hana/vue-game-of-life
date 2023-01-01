<template>
  <button @click="randomCell" :disabled="buttonStatus.random">random</button>
  <button @click="startLife" :disabled="buttonStatus.start">start</button>
  <button @click="stopLife" :disabled="buttonStatus.stop">stop</button>
  <button @click="resetField" :disabled="buttonStatus.reset">reset</button>
  <div id="playField">
    <p v-for="row in FIELD" v-bind:key="row.key">
      <span v-for="block in row.arr" v-bind:key="block.key">
        {{ block.icon }}
      </span>
    </p>
  </div>
</template>

<script setup>
// 初期設定
import { ref } from "vue";
// フィールドの設定
const FIELD_WIDTH = 30;
const FIELD_HEIGHT = 80;
const DENSITY = 40;
const SPEED = 100;
// フィールドの状態を管理するref
const FIELD = ref([]);
const futureFIELD = ref([]);
// ボタンの設定
const buttonStatus = ref({
  start: false,
  stop: true,
  random: false,
  reset: false,
});
// カーソルの位置
let cursorX = 0;
let cursorY = 0;
// インターバルID
let intervalId = null;

// keydownイベントの制御
// カーソルの移動
document.addEventListener("keydown", moveCursor, false);
function moveCursor(event) {
  switch (event.key) {
    case "ArrowUp":
      cursorY > 0 ? cursorY-- : cursorY;
      break;
    case "ArrowDown":
      cursorY < FIELD_HEIGHT - 1 ? cursorY++ : cursorY;
      break;
    case "ArrowRight":
      cursorX < FIELD_WIDTH - 1 ? cursorX++ : cursorX;
      break;
    case "ArrowLeft":
      cursorX > 0 ? cursorX-- : cursorX;
      break;
  }
  renderField();
}
// コマの生き死にを制御
document.addEventListener("keydown", toggleIcon, false);
function toggleIcon(event) {
  if (event.key === " ") {
    FIELD.value[cursorY].arr[cursorX].isAlive =
      !FIELD.value[cursorY].arr[cursorX].isAlive;
  }
}

// ランダムに生き死にを制御
function randomCell() {
  for (let y = 0; y < FIELD_HEIGHT; y++) {
    for (let x = 0; x < FIELD_WIDTH; x++) {
      let random = Math.floor(Math.random() * 100);
      if (random <= DENSITY) {
        FIELD.value[y].arr[x].isAlive = true;
      } else {
        FIELD.value[y].arr[x].isAlive = false;
      }
    }
  }
  renderField();
}

// ゲーム開始のボタン
function startLife() {
  intervalId = setInterval(decideDeadOrAlive, SPEED);
  buttonStatus.value.start = true;
  buttonStatus.value.stop = false;
  buttonStatus.value.random = true;
  buttonStatus.value.reset = true;
}
// 隣接するコマの生き死にを判定する関数
function getCountExistence(x, y) {
  let count = 0;
  for (let i = -1; i < 2; i++) {
    for (let j = -1; j < 2; j++) {
      let checkY = i + y;
      let checkX = j + x;
      if (i === 0 && j === 0) {
        continue;
      } else if (checkX < 0 || checkY < 0) {
        continue;
      } else if (checkX > FIELD_WIDTH - 1 || checkY > FIELD_HEIGHT - 1) {
        continue;
      }
      if (FIELD.value[checkY].arr[checkX].isAlive) {
        count++;
      }
    }
  }
  return count;
}
// セルの生き死にを判断する関数
function decideDeadOrAlive() {
  for (let y = 0; y < FIELD_HEIGHT; y++) {
    for (let x = 0; x < FIELD_WIDTH; x++) {
      if (FIELD.value[y].arr[x].isAlive) {
        if (getCountExistence(x, y) === 2 || getCountExistence(x, y) === 3) {
          futureFIELD.value[y].arr[x].isAlive = true;
        } else {
          futureFIELD.value[y].arr[x].isAlive = false;
        }
      } else if (!FIELD.value[y].arr[x].isAlive) {
        if (getCountExistence(x, y) === 3) {
          futureFIELD.value[y].arr[x].isAlive = true;
        }
      }
    }
  }
  console.log(FIELD.value === futureFIELD.value);
  for (let y = 0; y < FIELD_HEIGHT; y++) {
    for (let x = 0; x < FIELD_WIDTH; x++) {
      if (futureFIELD.value[y].arr[x].isAlive) {
        FIELD.value[y].arr[x].isAlive = true;
      } else {
        FIELD.value[y].arr[x].isAlive = false;
      }
    }
  }
  renderField();
}
// 停止ボタン
function stopLife() {
  clearInterval(intervalId);
  buttonStatus.value.start = false;
  buttonStatus.value.stop = true;
  buttonStatus.value.random = false;
  buttonStatus.value.reset = false;
}

resetField();
renderField();
// フィールドの初期化
function resetField() {
  for (let y = 0; y < FIELD_HEIGHT; y++) {
    FIELD.value[y] = { key: `y${y}`, arr: [] };
    futureFIELD.value[y] = { key: `y${y}`, arr: [] };
    for (let x = 0; x < FIELD_WIDTH; x++) {
      FIELD.value[y].arr[x] = {
        key: `x${x}y${y}`,
        icon: "・",
        isAlive: false,
      };
      futureFIELD.value[y].arr[x] = {
        key: `x${x}y${y}`,
        icon: "・",
        isAlive: false,
      };
    }
  }
}
// フィールドのレンダリング
function renderField() {
  // データに合わせてiconを変更
  for (let y = 0; y < FIELD_HEIGHT; y++) {
    for (let x = 0; x < FIELD_WIDTH; x++) {
      if (FIELD.value[y].arr[x].isAlive) {
        FIELD.value[y].arr[x].icon = "■";
      } else {
        FIELD.value[y].arr[x].icon = "・";
      }
    }
  }
  FIELD.value[cursorY].arr[cursorX].icon = "◎";
}
</script>

<style scoped>
p {
  font-size: 1rem;
  line-height: 6px;
  margin: 0;
  padding: 0;
}
span {
  margin: 0;
  padding: 0;
  line-height: 6px;
  font-size: 6px;
}
</style>
