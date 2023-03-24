<template>
    <div class="simple-keyboard"></div>
  </template>
  
  <script>
  import Keyboard from "simple-keyboard";
  import "simple-keyboard/build/css/index.css";
  import { onMounted, ref, watch } from "vue";
  
  export default {
    name: "SimpleKeyboard",
    props: {
      lettersGuessed: Object,
    },
    emits: ["onKeyPress"],
    setup(props, { emit }) {
      const keyboard = ref(null);
  
      const initializeKeyboard = {
        layout: {
          default: [
            "Q W E R T Y U I O P",
            "A S D F G H J K L",
            "{enter} Z X C V B N M {bksp}",
          ],
        },
        display: {
          "{bksp}": "⌫",
          "{enter}": "ENTER",
        },
        onKeyPress: (button) => {
          emit("onKeyPress", button);
        },
      };
  
      onMounted(() => {
        keyboard.value = new Keyboard("simple-keyboard", initializeKeyboard);
      });
  
      watch(
        () => props.lettersGuessed,
        (lettersGuessed, prevGuessedLetters) => {
          keyboard.value.addButtonTheme(lettersGuessed.miss.join(" "), "miss");
          keyboard.value.addButtonTheme(lettersGuessed.found.join(" "), "found");
          keyboard.value.addButtonTheme(lettersGuessed.hint.join(" "), "hint");
        },
        { deep: true }
      );
    },
  };
  </script>
  
  <style>
  div.miss {
    @apply bg-gray-500 !important;
    @apply text-white !important;
  }
  div.found {
    @apply bg-green-600 !important;
    @apply text-white !important;
  }
  div.hint:not(.found) {
    @apply bg-yellow-500 !important;
    @apply text-white !important;
  }
  </style>
  