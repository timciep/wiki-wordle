<template>
  <div>
    <div class="flex flex-wrap gap-1">
      <div v-for="letter, idx in word_status" :key="idx"
      :class="[
        letter.letter === ' ' ? 'w-full h-0' : ''
      ]">
        <div v-if="letter.letter !== ' '">
          <!-- <input type="text" value="x" class="w-2 absolute opacity-0"> -->

          <button v-if=" ! letter.free"
          @click="active_idx = idx"
          type="button"
          class="border-solid w-6 h-6 md:w-8 md:h-8 md:text-lg flex justify-center items-center"
          :class="[
            idx === active_idx ? 'border-4 border-blue-900' : 'border-2 border-gray-500',
            letter.guess === letter.letter ? 'bg-green-200' : '',
            letter.guess !== ' ' && letter.guess !== letter.letter ? 'bg-red-200' : '',
          ]">
            <div>
              {{ letter.guess }}
            </div>
          </button>

          <div v-else>
            {{ letter.letter }}
          </div>
        </div>
      </div>
    </div>

    <div v-cloak v-if="word.length && correctAnswer"
    class="mt-3 text-green-800 font-bold text-left">
      Nice! You got it!
    </div>

    <div class="mt-3 text-left">
      <button class="py-1 px-3 text-md rounded"
      style="background: #ececec;"
      @click="show_keyboard = ! show_keyboard">
        Keyboard
        {{ !show_keyboard ? '&or;' : '&and;' }}
      </button>

      <keyboard v-show="show_keyboard"
      @onKeyPress="handleKey"></keyboard>

    </div>

    <div class="mt-3 text-left flex gap-3">
      <button class="py-1 px-3 text-md rounded bg-blue-800 hover:bg-blue-500 text-white"
      v-show=" ! correctAnswer"
      type="button"
      @click="hint()">
        Hint
        <span class="text-xs">[SHIFT]</span>
      </button>

      <button type="button"
      v-show=" ! correctAnswer"
      @click="reveal()"
      class="rounded bg-gray-800 hover:bg-gray-500 text-white text-md py-1 px-3">
        Reveal
        <span class="text-xs">[ENTER]</span>
      </button>

      <button class="py-1 px-3 text-md rounded bg-green-800 hover:bg-green-500 text-white"
      type="button"
      @click="$emit('newWord')">
        New
        <span class="text-xs">[TAB]</span>
      </button>
    </div>
  </div>
</template>

<script>
import Keyboard from './Keyboard';
const _ = require('lodash');

export default {
  name: 'Word',

  props: {
    word: String
  },

  components: {
    Keyboard
  },

  data() {
    return {
      word_status: [],
      active_idx: 0,
      input_string: '',

      show_keyboard: false,
    }
  },

  methods: {
    refreshWordStatus() {
      this.word_status = _.reduce(this.word, function(result, letter) {
        const free = ! letter.match(/^[a-zA-Z]|[0-9]$/);

        result.push({
          letter: letter.toUpperCase(),
          guess: free ? letter : ' ',
          free: free,
        });
        
        return result;
      }, []);
    },

    handleKey(key) {
      // console.log(key);
      switch (key) {
        case 'Backspace':
        case '{bksp}':
          if (this.word_status[this.active_idx].guess === ' ') {
            this.prevLetter();
          } else {
            this.word_status[this.active_idx].guess = ' ';
          }
          break;
        case 'Enter':
          this.reveal();
          break;
        case 'ArrowLeft':
          this.prevLetter();
          break;
        case 'ArrowRight':
          this.nextLetter();
          break;
        case 'Tab':
          this.$emit('newWord');
          break;
        case 'Shift':
          this.hint();
          break;
        default:
          if (key.length === 1 && /^[a-zA-Z]|[0-9]$/.test(key)) {
            this.word_status[this.active_idx].guess = key.toUpperCase();
            this.nextLetter();
          }
      }
    },

    nextLetter() {
      this.active_idx = Math.min(this.word.length - 1, this.active_idx + 1);
      if (this.word_status[this.active_idx].free) {
        this.nextLetter();
      }
    },

    prevLetter() {
      this.active_idx = Math.max(0, this.active_idx - 1);
      if (this.word_status[this.active_idx].free) {
        this.prevLetter();
      }
    },

    reveal() {
      this.word_status.forEach(element => {
        element.guess = element.letter;
      });
    },

    hint() {
      for (let i=0; i < 3; i++) { // reveal 3 unrevealed letters.
        let letter = _.sample(_.filter(this.word_status, (_letter) => {
          return _letter.letter !== _letter.guess;
        }));

        if (letter) {
          letter.guess = letter.letter;
        }
      }
    }
  },

  computed: {
    guess_word: function() {
      return _.reduce(this.word_status, function(result, word) {
        return result + word.guess;
      }, '');
    },

    correctAnswer: function() {
      return this.word === this.guess_word;
    },
  },

  watch: {
    word: function() {
      this.refreshWordStatus();
    },

  },

  mounted() {
    this.refreshWordStatus();

    window.addEventListener('keydown', event => {
      event.preventDefault();
      this.handleKey(event.key);
    });
  },
}
</script>
