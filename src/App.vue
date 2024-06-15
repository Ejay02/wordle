<script setup>
import SimpleKeyboard from "./components/SimpleKeyboard.vue";
import WordRow from "./components/WordRow.vue";
import { reactive, onMounted, computed, watch } from "vue";
import { generate, count } from "random-words";
import confetti from "canvas-confetti";

const state = reactive({
  solution: generate({ minLength: 5, maxLength: 5 }),
  guesses: ["", "", "", "", "", ""],
  currentGuessIndex: 0,
  guessedLetters: {
    miss: [],
    found: [],
    hint: [],
  },
});


const wonGame = computed(
  () => state.guesses[state.currentGuessIndex - 1] === state.solution
);
const lostGame = computed(() => !wonGame.value && state.currentGuessIndex >= 6);

const handleInput = (key) => {
  if (state.currentGuessIndex >= 6 || wonGame.value) {
    return;
  }
  const currentGuess = state.guesses[state.currentGuessIndex];
  if (key == "{enter}") {
    // SEND GUESS
    if (currentGuess.length == 5) {
      state.currentGuessIndex++;
      setTimeout(() => {
        for (var i = 0; i < currentGuess.length; i++) {
          let c = currentGuess.charAt(i);
          if (c == state.solution.charAt(i)) {
            state.guessedLetters.found.push(c);
          } else if (state.solution.indexOf(c) != -1) {
            state.guessedLetters.hint.push(c);
          } else {
            state.guessedLetters.miss.push(c);
          }
        }
      }, 2500);
    }
  } else if (key == "{bksp}") {
    // REMOVE LAST LETTER
    state.guesses[state.currentGuessIndex] = currentGuess.slice(0, -1);
  } else if (currentGuess.length < 5) {
    // ADD LETTER IF ALPHABETICAL
    const alphaRegex = /[a-zA-Z]/;
    if (alphaRegex.test(key)) {
      state.guesses[state.currentGuessIndex] += key;
    }
  }
};

const resetGame = () => {
  window.location.reload();
};

watch(wonGame, (newValue) => {
  if (newValue) {
    var duration = 15 * 1000;
    var animationEnd = Date.now() + duration;
    var defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 };

    function randomInRange(min, max) {
      return Math.random() * (min - max) + min;
    }
    var interval = setInterval(function () {
      var timeLeft = animationEnd - Date.now();

      if (timeLeft <= 0) {
        return clearInterval(interval);
      }

      var particleCount = 50 * (timeLeft / duration);
      // since particles fall down, start a bit higher than random
      confetti({
        ...defaults,
        particleCount,
        origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
      });
      confetti({
        ...defaults,
        particleCount,
        origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
      });
    }, 250);
  }
});

onMounted(() => {
  window.addEventListener("keyup", (e) => {
    e.preventDefault();
    let key =
      e.keyCode == 13
        ? "{enter}"
        : e.keyCode == 8
        ? "{bksp}"
        : String.fromCharCode(e.keyCode).toLowerCase();
    handleInput(key);
  });
});
</script>

<template>
  <div class="pt-5">
    <div class="flex flex-col h-screen max-w-md mx-auto justify-evenly">
      <div>
        <word-row
          v-for="(guess, i) in state.guesses"
          :key="i"
          :value="guess"
          :solution="state.solution"
          :submitted="i < state.currentGuessIndex"
        />
      </div>
      <p v-if="wonGame" class="text-center">🏆 Congratulation! you solved it.</p>
      <div v-else-if="lostGame" class="text-center">
        <p class="text-center">😔 Out of tries.</p>
        <p>
          The correct word :
          <span class="bg-cyan-500 p-1 rounded m-2">
            {{ state.solution }}
          </span>
        </p>
      </div>
      <simple-keyboard
        @onKeyPress="handleInput"
        :guessedLetters="state.guessedLetters"
      />

      <button
        v-if="wonGame || lostGame"
        @click="resetGame"
        class="mt-4 p-2 bg-cyan-500 text-white rounded"
      >
        Play Again
      </button>
    </div>
  </div>
</template>

<style></style>
