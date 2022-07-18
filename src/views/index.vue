<template>
  <div style="text-align: center">
    Minesweeper
    <div style="padding: 5px">
      <div
        v-for="(row, y) in state"
        :key="y"
        style="display: flex; align-blocks: center; justify-content: center"
      >
        <button
          v-for="(block, x) in row"
          :key="x"
          @click="onClick(block)"
          :class="getBlockClass(block)"
          style="border: 1px solid; border-color: gray"
          @contextmenu.prevent="onRightClick(block)"
        >
          <template v-if="block.flagged">
            <div style="color: red">
              <Icon icon="mdi:flag" />
            </div>
          </template>
          <template
            v-else-if="block.revealed || dev"
            style="display: flex; align-blocks: center"
          >
            <div v-if="block.mine">
              <Icon icon="mdi:mine" />
            </div>
            <div v-else>{{ block.adjacentMines }}</div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { reactive } from "@vue/reactivity";
import { Icon } from "@iconify/vue";
import { watchEffect } from "@vue/runtime-core";
import { ref } from "vue";
// import func from "vue-temp/vue-editor-bridge";
// import func from "vue-temp/vue-editor-bridge";
import { BlockState } from "../styles/types";

const WIDTH = 10;
const HEIGHT = 10;
const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from(
      { length: WIDTH },
      (_, x) => <BlockState>{ x, y, adjacentMines: 0, revealed: false }
    )
  )
);

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
  "text-transparent",
  "text-blue-500",
  "text-green-500",
  "text-yellow-500",
  "text-orange-500",
  "text-red-500",
  "text-purple-500",
  "text-pink-500",
  "text-teal-500",
];

function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine) return;
      getSiblings(block).forEach((b) => {
        if (b.mine) block.adjacentMines += 1;
      });
    });
  });
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
let dev = false;

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block);
    mineGenerated = true;
  }
  block.revealed = true;
  if (block.mine) alert("BOOOM!!");
  expendZero(block);
}

function onRightClick(block: BlockState) {
  if (block.revealed) return;
  block.flagged = !block.flagged;
}

function getBlockClass(block: BlockState) {
  if (block.flagged) return "";
  if (!block.revealed) return "";
  return block.mine ? "text-red" : numberColors[block.adjacentMines];
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

watchEffect(checkGameState);

function checkGameState() {
  if (!mineGenerated) return;
  const blocks = state.value.flat();

  if (blocks.every((block) => block.revealed || block.flagged)) {
    if (blocks.some((block) => block.flagged && !block.mine)) {
      alert("You cheat!");
    } else {
      alert("YOU Wiiin!!");
    }
  }
}
</script>

<style>
button {
  padding: 0;
  width: 30px;
  height: 30px;
}
button:hover {
  background-color: gray;
}
.text-red {
  color: red;
  background-color: rgba(121, 50, 50, 0.5);
}
.text-transparent {
  color: transparent;
  background-color: lightgray;
  border: 1px solid black;
}
.text-blue-500 {
  color: blue;
  background-color: lightgray;
  border: 1px solid black;
}
.text-green-500 {
  color: green;
  background-color: lightgray;
  border: 1px solid black;
}
.text-yellow-500 {
  color: yellow;
  background-color: lightgray;
  border: 1px solid black;
}
.text-orange-500 {
  color: orange;
  background-color: lightgray;
  border: 1px solid black;
}
.text-red-500 {
  color: darkred;
  background-color: lightgray;
  border: 1px solid black;
}
.text-purple-500 {
  color: purple;
  background-color: lightgray;
  border: 1px solid black;
}
.text-pink-500 {
  color: pink;
  background-color: lightgray;
  border: 1px solid black;
}
.text-teal-500 {
  color: teal;
  background-color: lightgray;
  border: 1px solid black;
}
</style>