<template>
  
  <div class="flex flex-col h-screen max-w-md mx-auto justify-evenly">
    <h1 class="title-center wordle-title">Wordle</h1>
    <div>
      <WordRow
        v-for="(guess, index) in state.guesses"
        :key="index"
        :value="guess"
        :solution="state.solution"
        :submitted="index < state.currentGuessIndex"
      />
    </div>
    <p v-if="gameWon" class="text-center">
      Congratulations! You won!
    </p>

    <p v-else-if="gameLost" class="text-center">
      :( You lost.
    </p>
    <SimpleKeyboard
      @onKeyPress="handleInput"
      :lettersGuessed="state.lettersGuessed"
      />
  </div>
</template>

<script>
import SimpleKeyboard from "./components/SimpleKeyboard.vue";
import WordRow from "./components/WordRow.vue";
import { reactive, onMounted, computed } from "vue";

export default {
  components: {
    SimpleKeyboard,
    WordRow,
  },
  setup() {
    const state = reactive({
      solution: "",
      guesses: ["","","","","",""],
      currentGuessIndex: 0,
      lettersGuessed: {
        miss:[],
        found:[],
        hint:[],
      }
    });

    const gameWon = computed(
      () =>
        state.guesses[state.currentGuessIndex - 1] === state.solution
    );

    const gameLost = computed(() => !gameWon.value && state.currentGuessIndex >= 6);
    
    // Fetch random word from wordle_words.csv
    const fetchRandomWord = async () => {
      const response = await fetch("/wordle_words.csv");
      const text = await response.text();
      const words = text.split("\n");
      const randomIndex = Math.floor(Math.random() * words.length);
      state.solution = words[randomIndex].trim().toUpperCase();
    };


    const handleInput = (key) => {
      if (state.currentGuessIndex >= 6 || gameWon.value) {
        return;
      }
      const currentGuess = state.guesses[state.currentGuessIndex];

      if (key == "{enter}") {
        // send guess.
        if (currentGuess.length === 5) {
          state.currentGuessIndex++;
          for(let i = 0; i < currentGuess.length; i++) {
            // Will check if the letter is in the solution.
            let guessedLetter = currentGuess.charAt(i);
            // If the letter is in the solution, add it to the found array.
            if (guessedLetter == state.solution.charAt(i)) {
              state.lettersGuessed.found.push(guessedLetter);
            }
            // If the letter is in the solution, but not in the correct position, add it to the hint array.
            else if (state.solution.indexOf(guessedLetter) != -1) {
              state.lettersGuessed.hint.push(guessedLetter);
            }
            // If the letter is not in the solution, add it to the miss array.
            else {
              state.lettersGuessed.miss.push(guessedLetter);
            }
          }
        }
      } else if (key == "{bksp}"){
        // remove last letter.
        state.guesses[state.currentGuessIndex] = currentGuess.slice(0, -1);
      } else if (currentGuess.length < 5) {
        // add letter.
        const alphaRegex = /[a-zA-Z]/;
        if (alphaRegex.test(key)) {
          state.guesses[state.currentGuessIndex] += key;
        }
      }
    };


    onMounted(async () => {
        await fetchRandomWord();

        window.addEventListener("keyup", (e) => {
          e.preventDefault();
          let key = e.key == 13
            ? "{enter}"
            : e.key == 8 ? "{bksp}"
            : String.fromCharCode(e.key).toLocaleLowerCase();
          handleInput(key);
        });
      });

      return {
        state,
        gameWon,
        gameLost,
        handleInput,
      };
    },
  };
</script>

<style scoped>
.title-center {
  text-align: center;
  font-size: 30px;
}

.wordle-title {
  font-size: 60px;
  font-weight: bold;
  color: #22c55e;
  text-shadow: 3px 3px 0 #000,
               -1px -1px 0 #000,  
               1px -1px 0 #000,
               -1px 1px 0 #000,
               1px 1px 0 #000;
}
</style>