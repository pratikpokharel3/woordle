<script setup lang="ts">
import { ref } from "vue";
import { Board } from "./interfaces";
import { answers } from "./words";

import Appbar from "./Appbar.vue";
import Keyboard from "./Keyboard.vue";

const board = ref<Board[][]>([]);
let keypadWords = ref<Board[]>([]);
const shakeTileState = ref(false);

let enteredWord: string[] = [];
let boardRowIdx = 0;
let boardColIdx = 0;
let hasUserWon = false;
let wordOfTheDay = "";

const keypads = [
  "QWERTYUIOP".split(""),
  "ASDFGHJKL".split(""),
  "ZXCVBNM".split(""),
];

function initializeBoard() {
  board.value = Array.from({ length: 5 }, () =>
    Array.from({ length: 5 }, () => ({ letter: "", state: -1 }))
  );
}

initializeBoard();
setWordOfTheDay();

function setWordOfTheDay() {
  const idx = Math.floor(Math.random() * answers.length);
  wordOfTheDay = answers[idx].toUpperCase();

  if (import.meta.env.DEV) {
    console.log(wordOfTheDay);
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

  board.value[boardRowIdx][boardColIdx].letter = letter;
  boardColIdx++;
  enteredWord.push(letter);
}

function removeLetterFromBoard() {
  if (boardColIdx === 0) {
    return;
  }

  boardColIdx--;
  board.value[boardRowIdx][boardColIdx].letter = "";
  enteredWord.pop();
}

function handleKeypadsWords(letter: string, state: number) {
  const idx = keypadWords.value.findIndex((val) => val.letter === letter);

  if (idx !== -1) {
    if (keypadWords.value[idx].state !== 1)
      keypadWords.value[idx] = {
        letter,
        state,
      };
    return;
  }

  keypadWords.value.push({ letter, state });
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

  let correctPosIdx: number[] = [];
  let inCorrectPosIdx: number[] = [];

  // Check if any letter is in correct position
  for (let i = 0; i < 5; i++) {
    if (wordOfTheDay[i] === word[i]) {
      board.value[boardRowIdx][i].state = 1;
      correctPosIdx.push(i);
      handleKeypadsWords(word[i], 1);
    }
  }

  //Check if any letter is in incorrect position
  for (let i = 0; i < 5; i++) {
    if (correctPosIdx.includes(i)) {
      continue;
    }

    for (let j = 0; j < 5; j++) {
      if (correctPosIdx.includes(j) || inCorrectPosIdx.includes(j)) {
        continue;
      }

      if (wordOfTheDay[i] === word[j]) {
        board.value[boardRowIdx][j].state = 2;
        inCorrectPosIdx.push(j);
        handleKeypadsWords(word[j], 2);
        break;
      }
    }
  }

  //Check if any letter is not in the word
  for (let i = 0; i < 5; i++) {
    if (correctPosIdx.includes(i) || inCorrectPosIdx.includes(i)) {
      continue;
    }

    board.value[boardRowIdx][i].state = 3;
    handleKeypadsWords(word[i], 3);
  }

  hasUserWon = word === wordOfTheDay;

  if (hasUserWon) {
    displayResult();
    return;
  }

  //Check if the number of attempts is greater than 5
  if (boardRowIdx === 4) {
    displayResult();
    return;
  }

  enteredWord = [];
  boardRowIdx++;
  boardColIdx = 0;
  correctPosIdx = [];
  inCorrectPosIdx = [];
}

function displayResult() {
  let message = "";

  if (hasUserWon) {
    if (boardRowIdx === 0) {
      message = "WELL DONE! YOU ARE THE CHOSEN ONE!!!";
    } else if (boardRowIdx === 1) {
      message = "IMPRESSIVE! When did you became so good at this!!!";
    } else if (boardRowIdx === 2) {
      message = "GOOD! You are a smart person, I get it!!!";
    } else if (boardRowIdx === 3) {
      message = "NOT BAD! Not bad at all!!!";
    } else {
      message = "WOW! You finally got it at the end, cheers!!!";
    }
  } else {
    message =
      "YOU LOST! Do you even know how to play this game!!!\nWord of the Day: " +
      wordOfTheDay;
  }

  setTimeout(() => {
    alert(message);

    enteredWord = [];
    boardRowIdx = 0;
    boardColIdx = 0;
    hasUserWon = false;
    keypadWords.value = [];

    initializeBoard();
    setWordOfTheDay();
  }, 500);
}
</script>

<template>
  <div class="toast_alert" v-if="shakeTileState && boardColIdx < 5">
    <div>Not enough letters!</div>
  </div>

  <div class="container">
    <Appbar></Appbar>

    <div class="row g-0 justify-content-center">
      <div class="col-9 col-md-7 col-lg-5 col-xl-4">
        <div class="board" v-for="(row, index) in board">
          <div
            class="tile"
            :class="[
              { entered: letter !== '' },
              { correct_pos: state === 1 },
              { incorrect_pos: state === 2 },
              { not_included: state === 3 },
              { tile_shake: shakeTileState && index === boardRowIdx },
            ]"
            v-for="{ letter, state } in row"
          >
            {{ letter }}
          </div>
        </div>
      </div>
    </div>

    <Keyboard
      :keypads="keypads"
      :keypad-words="keypadWords"
      @getterLetter="getLetter"
    ></Keyboard>
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

.correct_pos {
  color: white;
  border: 2px solid #6aaa64;
  background-color: #6aaa64;
}

.incorrect_pos {
  color: white;
  border: 2px solid #c9b458;
  background-color: #c9b458;
}

.not_included {
  color: white;
  border: 2px solid #787c7e;
  background-color: #787c7e;
}
</style>
