<template>
  <div :class="keyboardClass"></div>
</template>

<script>
// Ref: https://hodgef.com/simple-keyboard/
import Keyboard from "simple-keyboard";
import "simple-keyboard/build/css/index.css";

export default {
  name: "Keyboard",

  props: {
    keyboardClass: {
      default: "simple-keyboard",
      type: String
    },
  },

  data: () => ({
    keyboard: null
  }),

  mounted() {
    this.keyboard = new Keyboard(this.keyboardClass, {
      onChange: this.onChange,
      onKeyPress: this.onKeyPress,

      layout: {
         'default': [
            '1 2 3 4 5 6 7 8 9 0 {bksp}',
            'Q W E R T Y U I O P',
            'A S D F G H J K L ',
            ' Z X C V B N M  ',
         ],
      },

      display: {
         '{bksp}': '&larrb;',
      }
    });
  },
  methods: {
    onKeyPress(button) {
      this.$emit("onKeyPress", button);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
