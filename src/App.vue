<script setup lang="ts">
import { ref } from "vue";
import { answers } from "./words";

interface Board {
  value: string;
  state: number;
}

const boardArr = ref(new Array(5));
const shakeTileState = ref(false);

let enteredWord: string[] = [];
let boardRowIdx = 0;
let boardColIdx = 0;
let a: number[] = [];
let b: number[] = [];
let w: string[] = [];
let hasUserWon = false;
let wordOfTheDay = "";

const keypads = [
  "QWERTYUIOP".split(""),
  "ASDFGHJKL".split(""),
  "ZXCVBNM".split("")
];

setWordOfTheDay();
initializeBoard();

function setWordOfTheDay() {
  const idx = Math.floor(Math.random() * answers.length);
  wordOfTheDay = answers[idx].toUpperCase();

  if (import.meta.env.DEV) {
    console.log(wordOfTheDay);
  }
}

function initializeBoard() {
  for (let i = 0; i < 5; i++) {
    boardArr.value[i] = new Array<Board>(5);

    for (let j = 0; j < 5; j++) {
      boardArr.value[i][j] = {
        value: "",
        state: -1
      };
    }
  }
}

function getLetter(letter: string) {
  if (letter === "ENTER") {
    compareLetters();
  } else if (letter === "CLEAR") {
    removeLetterFromBoard();
  } else {
    addLetterToBoard(letter);
  }
}

function addLetterToBoard(letter: string) {
  if (boardColIdx > 4) {
    return;
  }

  boardArr.value[boardRowIdx][boardColIdx].value = letter;
  boardColIdx++;
  enteredWord.push(letter);
}

function removeLetterFromBoard() {
  if (boardColIdx === 0) {
    return;
  }

  boardColIdx--;
  boardArr.value[boardRowIdx][boardColIdx].value = "";
  enteredWord.pop();
}

function compareLetters() {
  if (boardColIdx !== 5) {
    shakeTileState.value = true;

    setTimeout(() => {
      shakeTileState.value = false;
    }, 1000);
    return;
  }

  const word = enteredWord.join("");

  //Check if any letter is in correct position
  for (let i = 0; i < 5; i++) {
    for (let j = 0; j < 5; j++) {
      if (wordOfTheDay[i] === word[j]) {
        if (i === j) {
          boardArr.value[boardRowIdx][j].state = 1;
          a.push(j);
          w.push(wordOfTheDay[j]);
        }
      }
    }
  }

  //Check if any letter is in incorrect position
  for (let i = 0; i < 5; i++) {
    if (a.includes(i)) {
      continue;
    }

    for (let j = 0; j < 5; j++) {
      if (wordOfTheDay[i] === word[j]) {
        if (i !== j) {
          if (a.includes(j) || b.includes(j)) {
            continue;
          } else {
            boardArr.value[boardRowIdx][j].state = 2;
            b.push(j);
            break;
          }
        }
      }
    }
  }

  //Check if any letter is not in the word
  for (let i = 0; i < 5; i++) {
    if (a.includes(i) || b.includes(i)) {
      continue;
    }

    boardArr.value[boardRowIdx][i].state = 3;
  }

  hasUserWon = w.join("") === wordOfTheDay;

  if (hasUserWon) {
    displayResult();
    return;
  }

  //Check if the number of attempts is greater than 5
  if (boardRowIdx === 4) {
    calculateResult();
    return;
  }

  enteredWord = [];
  boardRowIdx++;
  boardColIdx = 0;
  a = [];
  b = [];
  w = [];
}

function calculateResult() {
  if (hasUserWon) {
    displayResult();
  } else {
    displayResult();
  }
}

function displayResult() {
  let message = "";

  if (hasUserWon) {
    if (boardRowIdx === 0) {
      message = "WELL DONE! YOU ARE THE CHOSEN ONE!!!";
    } else if (boardRowIdx === 1) {
      message = "IMPRESSIVE! When did you became so good at this!!!";
    } else if (boardRowIdx === 2) {
      message = "GOOD! You are a very smart person!!!";
    } else if (boardRowIdx === 3) {
      message = "NOT BAD! Not bad at all!!!";
    } else {
      message = "WOW! You finally got it at the end, congrats!!!";
    }
  } else {
    message = "YOU LOST! Do you even know how to play this game!!!";
  }

  setTimeout(() => {
    alert(message);

    enteredWord = [];
    boardRowIdx = 0;
    boardColIdx = 0;
    a = [];
    b = [];
    w = [];
    hasUserWon = false;

    initializeBoard();
    setWordOfTheDay();
  }, 500);
}
</script>

<template>
  <div class="toast_alert" v-if="shakeTileState && boardColIdx < 5">
    <div>Not enough letters!</div>
  </div>

  <nav class="navbar">
    <div class="container">
      <div class="navbar-brand fw-bold">Woordle</div>

      <a href="https://github.com/pratikpokharel3/woordle" target="_blank">
        <svg
          class="github_svg"
          viewBox="0 0 24 24"
          xmlns="http://www.w3.org/2000/svg"
        >
          <title>github</title>
          <path
            d="M12,2A10,10 0 0,0 2,12C2,16.42 4.87,20.17 8.84,21.5C9.34,21.58 9.5,21.27 9.5,21C9.5,20.77 9.5,20.14 9.5,19.31C6.73,19.91 6.14,17.97 6.14,17.97C5.68,16.81 5.03,16.5 5.03,16.5C4.12,15.88 5.1,15.9 5.1,15.9C6.1,15.97 6.63,16.93 6.63,16.93C7.5,18.45 8.97,18 9.54,17.76C9.63,17.11 9.89,16.67 10.17,16.42C7.95,16.17 5.62,15.31 5.62,11.5C5.62,10.39 6,9.5 6.65,8.79C6.55,8.54 6.2,7.5 6.75,6.15C6.75,6.15 7.59,5.88 9.5,7.17C10.29,6.95 11.15,6.84 12,6.84C12.85,6.84 13.71,6.95 14.5,7.17C16.41,5.88 17.25,6.15 17.25,6.15C17.8,7.5 17.45,8.54 17.35,8.79C18,9.5 18.38,10.39 18.38,11.5C18.38,15.32 16.04,16.16 13.81,16.41C14.17,16.72 14.5,17.33 14.5,18.26C14.5,19.6 14.5,20.68 14.5,21C14.5,21.27 14.66,21.59 15.17,21.5C19.14,20.16 22,16.42 22,12A10,10 0 0,0 12,2Z"
          />
        </svg>
      </a>
    </div>
  </nav>

  <div class="container">
    <div class="row g-0 justify-content-center">
      <div class="col-9 col-md-7 col-lg-5 col-xl-4">
        <!-- tslint:disable-next-line -->
        <div class="board" v-for="(row, index) in boardArr">
          <div
            class="tile"
            :class="[
              { entered: tile.value !== '' },
              { correct_pos: tile.state === 1 },
              { incorrect_pos: tile.state === 2 },
              { not_included: tile.state === 3 },
              {
                tile_shake: shakeTileState && index === boardRowIdx
              }
            ]"
            v-for="tile in row"
          >
            {{ tile.value }}
          </div>
        </div>
      </div>
    </div>

    <div class="row g-0 justify-content-center mt-5">
      <div class="col-12 col-lg-8 col-xl-7 keyboard">
        <div class="keypad" v-for="i in keypads[0]" @click="getLetter(i)">
          {{ i }}
        </div>
      </div>

      <div class="col-12 col-lg-8 col-xl-7 mt-3 keyboard">
        <div class="keypad" v-for="i in keypads[1]" @click="getLetter(i)">
          {{ i }}
        </div>
      </div>

      <div class="col-12 col-lg-8 col-xl-7 mt-3 keyboard">
        <div class="keypad" @click="getLetter('CLEAR')">
          <svg
            viewBox="0 0 24 24"
            xmlns="http://www.w3.org/2000/svg"
            style="width: 20px; height: 20px"
          >
            <title>close</title>
            <path
              d="M5,15.59L6.41,17L10,13.41L13.59,17L15,15.59L11.41,12L15,8.41L13.59,7L10,10.59L6.41,7L5,8.41L8.59,12L5,15.59M2,3A2,2 0 0,0 0,5V19A2,2 0 0,0 2,21H17C17.69,21 18.23,20.64 18.59,20.11L24,12L18.59,3.88C18.23,3.35 17.69,3 17,3H2M2,5H17L21.72,12L17,19H2V5Z"
            />
          </svg>
        </div>

        <div class="keypad" v-for="i in keypads[2]" @click="getLetter(i)">
          {{ i }}
        </div>

        <div class="keypad" @click="getLetter('ENTER')">
          <svg
            viewBox="0 0 24 24"
            xmlns="http://www.w3.org/2000/svg"
            style="width: 20px; height: 20px"
          >
            <title>Enter</title>
            <path
              d="M20 4V10.5C20 14.09 17.09 17 13.5 17H7.83L10.92 20.09L9.5 21.5L4 16L9.5 10.5L10.91 11.91L7.83 15H13.5C16 15 18 13 18 10.5V4H20Z"
            />
          </svg>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
html,
body,
#app {
  height: 100%;
}

.toast_alert {
  top: 0.5rem;
  width: 100%;
  display: flex;
  position: absolute;
  justify-content: center;
}

.toast_alert div {
  color: white;
  padding: 0.5rem;
  font-size: 0.875rem;
  border-radius: 0.5rem;
  background-color: #000;
}

.github_svg {
  width: 24px;
  height: 24px;
  fill: #586774;
}

.github_svg:hover {
  cursor: pointer;
  fill: #000;
}

.board {
  gap: 0.5rem;
  display: flex;
  margin-top: 0.5rem;
  justify-content: center;
}

.tile {
  height: 3rem;
  width: 3.5rem;
  display: grid;
  place-items: center;
  border: 1px solid #dfdfdf;
}

.entered {
  border: 2px solid #787c7e;
  animation: entered_anim 0.2s forwards;
}

@keyframes entered_anim {
  0% {
    transform: scale(1.2);
  }

  100% {
    transform: scale(1);
  }
}

.tile_shake {
  animation: shake_anim 0.5s;
}

@keyframes shake_anim {
  0% {
    transform: translate(1px, 1px) rotate(0deg);
  }
  10% {
    transform: translate(-1px, -2px) rotate(-1deg);
  }
  20% {
    transform: translate(-3px, 0px) rotate(1deg);
  }
  30% {
    transform: translate(3px, 2px) rotate(0deg);
  }
  40% {
    transform: translate(1px, -1px) rotate(1deg);
  }
  50% {
    transform: translate(-1px, 2px) rotate(-1deg);
  }
  60% {
    transform: translate(-3px, 1px) rotate(0deg);
  }
  70% {
    transform: translate(3px, 1px) rotate(-1deg);
  }
  80% {
    transform: translate(-1px, -1px) rotate(1deg);
  }
  90% {
    transform: translate(1px, 2px) rotate(0deg);
  }
  100% {
    transform: translate(1px, -2px) rotate(-1deg);
  }
}

.keyboard {
  gap: 0.5rem;
  display: flex;
  justify-content: center;
}

.keypad {
  width: 3.5rem;
  padding: 1rem 0;
  font-weight: bold;
  text-align: center;
  font-size: 0.875rem;
  border-radius: 0.25rem;
  background-color: #d3d6da;
}

.keypad:hover {
  cursor: pointer;
}

.correct_pos {
  color: white;
  border: 2px solid #6aaa64;
  background-color: #6aaa64;
}

.incorrect_pos {
  color: white;
  border: 2px solid orange;
  background-color: orange;
}

.not_included {
  color: white;
  border: 2px solid #787c7e;
  background-color: #787c7e;
}
</style>
