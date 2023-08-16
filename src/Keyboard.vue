<script setup lang="ts">
import { Board } from "./interfaces";

const props = defineProps<{
  keypads: string[][];
  keypadWords: Board[];
}>();

defineEmits<{
  getterLetter: [letter: string];
}>();

function setKeyBoardColor(letter: string) {
  for (let i = 0; i < props.keypadWords.length; i++) {
    if (props.keypadWords[i].letter === letter) {
      if (props.keypadWords[i].state === 1) return "correct";
      else if (props.keypadWords[i].state === 2) return "incorrect";
      else if (props.keypadWords[i].state === 3) return "not_included";
    }
  }
}
</script>

<template>
  <div class="row g-0 justify-content-center mt-5">
    <div class="col-12 col-lg-8 col-xl-7 keyboard">
      <div
        class="keypad"
        :class="setKeyBoardColor(letter)"
        v-for="letter in props.keypads[0]"
        @click="$emit('getterLetter', letter)"
      >
        {{ letter }}
      </div>
    </div>

    <div class="col-12 col-lg-8 col-xl-7 mt-3 keyboard">
      <div
        class="keypad"
        v-for="letter in props.keypads[1]"
        :class="setKeyBoardColor(letter)"
        @click="$emit('getterLetter', letter)"
      >
        {{ letter }}
      </div>
    </div>

    <div class="col-12 col-lg-8 col-xl-7 mt-3 keyboard">
      <div class="keypad" @click="$emit('getterLetter', 'ENTER')">
        <svg
          viewBox="0 0 24 24"
          xmlns="http://www.w3.org/2000/svg"
          style="width: 20px; height: 20px"
        >
          <title>enter</title>
          <path
            d="M20 4V10.5C20 14.09 17.09 17 13.5 17H7.83L10.92 20.09L9.5 21.5L4 16L9.5 10.5L10.91 11.91L7.83 15H13.5C16 15 18 13 18 10.5V4H20Z"
          />
        </svg>
      </div>

      <div
        class="keypad"
        :class="setKeyBoardColor(letter)"
        v-for="letter in props.keypads[2]"
        @click="$emit('getterLetter', letter)"
      >
        {{ letter }}
      </div>

      <div class="keypad" @click="$emit('getterLetter', 'CLEAR')">
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
    </div>
  </div>
</template>

<style scoped>
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

.correct {
  color: white;
  background-color: #6aaa64;
}

.incorrect {
  color: white;
  background-color: #c9b458;
}

.not_included {
  border: none;
  color: white;
  background-color: #787c7e;
}
</style>
