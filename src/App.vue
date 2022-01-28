<template>
  <div id="app">
    <div class="max-w-screen-sm mx-auto px-2 mb-6 min-h-screen">
      <div class="mt-3 text-3xl italic">
        WikiWordle
      </div>

      <div v-if="loading"
      class="mt-3">
        Loading...
      </div>

      <div v-else>
        <div class="mt-3 text-center">
          <word :word="clean_word"
          @newWord="newWord()"></word>
        </div>

        <div class="mt-4">
          <img v-if="word_image.length"
          :src="word_image" alt="?" class="float-left pr-3 pb-3"
          style="max-width: 10rem;">

          <div v-html="clean_word_summary"></div>
        </div>

        <div class="mt-1">
          <a :href="word_url" target="_blank"
          class="text-blue-800 hover:text-blue-500 text-sm">
            Full Article
            &nearr;
          </a>
        </div>
      </div>
    </div>

    <div class="text-sm text-center mt-12 mb-5">
      Created by
      <a href="https://www.timcieplowski.com/" target="_blank"
      class="text-blue-800 hover:text-blue-500">
        Tim Cieplowski
        &nearr;
      </a>
    </div>
  </div>
</template>

<script>
import wiki from 'wikijs';

import Word from './components/Word.vue';

export default {
  name: 'App',

  components: {
    Word,
  },

  data() {
    return {
      word: '',
      word_summary: '',
      word_image: '',
      word_url: '',

      loading: true,
    }
  },

  methods: {
    async newWord() {
      this.loading = true;

      const word_result = await wiki({
        apiUrl: 'https://en.wikipedia.org/w/api.php'
      }).api({
        list: 'mostviewed',
        pvimlimit: 1,
        pvimoffset: Math.floor(Math.random() * 999) + 1,
      }).then(res => {
        return res.query.mostviewed.map(({ title, count }) => ({ title, count }));
      }).catch( async function(err) {
        console.error(err);
      });

      console.log({result: word_result});

      if (word_result.length) {
        this.word = word_result[0].title;
      } else {
        const rando_result = await wiki({
          apiUrl: 'https://en.wikipedia.org/w/api.php'
        }).random();

        this.word = rando_result[0];
      }

      const Page = await wiki({
        apiUrl: 'https://en.wikipedia.org/w/api.php'
      }).page(this.word);

      this.word_url = Page.fullurl;

      Promise.all([
        Page.summary(),
        Page.mainImage(),
      ]).then((values) => {
        // If word does not have a summary, get another word.
        if ( ! values[0].length ) {
          this.newWord();
        }

        this.word_summary = values[0] ?? '';
        this.word_image = values[1] ?? '';

        this.loading = false;
      });
    }
  },

  computed: {
    clean_word_summary: function() {
      // Replace _not letter or number_ with "|".
      const words_or = this.clean_word.replace(/[^A-Za-z0-9]/g, '|');
      return this.word_summary.replace(new RegExp(words_or, 'ig'), (match) => {
        return '&FilledSmallSquare;'.repeat(match.length);
      });
    },

    clean_word: function() {
      return this.word.replace(/( *\([^)]*\) *|\.)/g, '').toUpperCase();
    }
  },

  mounted() {
    this.newWord();
  },
}
</script>


