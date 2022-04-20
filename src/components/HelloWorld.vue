<script setup lang="ts">
import { BlockState } from "@/types";
import { reactive, ref } from "@vue/reactivity";
import { watchEffect } from "@vue/runtime-core";

const WIDTH = 12;
const HEIGHT = 12;
const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from(
      { length: WIDTH },
      (_, x): BlockState => ({ y, x, adjacentMines: 0, revealed: false })
    )
  )
);
// 生成炸弹
function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1) continue;
      if (Math.abs(initial.y - block.y) <= 1) continue;
      block.mine = Math.random() < 0.2;
    }
  }
  updateNumbers();
}

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
];

const numberColors = [
  "zero",
  "one",
  "two",
  "three",
  "four",
  "five",
  "six",
  "seven",
  "eight",
];

function updateNumbers() {
  state.value.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine) return;
      getSiblings(block).forEach((s) => {
        if (s.mine) block.adjacentMines++;
      });
    });
  });
}

function getSiblings(block: BlockState) {
  return directions
    .map(([dx, dy]) => {
      const x2 = block.x + dx;
      const y2 = block.y + dy;
      if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT) return undefined;
      return state.value[y2][x2];
    })
    .filter(Boolean) as BlockState[];
}

function getBlockClass(block: BlockState) {
  if (!block.revealed) {
    return "not-revealed";
  }
  return block.mine ? "mine" : numberColors[block.adjacentMines];
}

function expendZero(block: BlockState) {
  if (block.adjacentMines) return;

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true;
      expendZero(s);
    }
  });
}

let mineGenerated = false;
// 全部显示
const dev = false;

function onRightClick(block: BlockState) {
  block.flagged = !block.flagged;
}

function onClick(e: MouseEvent, block: BlockState) {
  console.log(e);
  if (!mineGenerated) {
    generateMines(block);
    mineGenerated = true;
  }
  block.revealed = true;
  if (block.mine) {
    alert("You lose!");
    state.value.forEach((row) => {
      row.forEach((block) => {
        block.revealed = true;
      });
    });
  }
  expendZero(block);
}

watchEffect(checkGameState);

// 检查游戏状态
function checkGameState() {
  const blocks = state.value.flat();
  const unrevealed = blocks.filter((block) => !block.revealed);
  const mines = blocks.filter((block) => block.mine);
  if (unrevealed.length === mines.length) {
    alert("You win!");
  }
}
</script>

<template>
  <div>
    <h1 style="margin: 20px">Minesweeper</h1>
    <div class="asd" v-for="(row, y) in state" :key="y">
      <button
        v-for="(block, x) in row"
        :key="x"
        :class="getBlockClass(block)"
        @click="onClick($event, block)"
        @contextmenu.prevent="onRightClick(block)"
      >
        <template v-if="block.flagged && !block.revealed">
          <div class="iconfont icon-flag_fill"></div>
        </template>
        <template v-if="block.revealed || dev">
          <div v-if="block.mine" class="iconfont icon-zhadan1"></div>
          <div class="font" v-else>
            {{ block.adjacentMines }}
          </div>
        </template>
      </button>
    </div>
  </div>
</template>

<style>
.asd {
  display: flex;
  justify-content: center;
  align-items: center;
}

button {
  width: 50px;
  height: 50px;
  border: 1px solid gray;
  display: flex;
  justify-content: center;
  align-items: center;
}

.iconfont.icon-flag_fill {
  color: red;
  font-size: 30px;
}

/* button:hover {
  background: rgb(144, 144, 224);
} */

button[class="mine"] {
  background-color: red;
}

.not-revealed {
  background: rgb(163, 162, 196);
}
.font {
  font-size: 20px;
  font-weight: bold;
}
.zero .font {
  opacity: 0;
}
.one {
  color: blue;
}
.two {
  color: green;
}
.three {
  color: red;
}
.four {
  color: purple;
}
.five {
  color: orange;
}
.six {
  color: yellow;
}
.seven {
  color: brown;
}
.eight {
  color: black;
}
</style>
