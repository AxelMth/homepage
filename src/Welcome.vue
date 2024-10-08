<script setup lang="ts">
import { ref, onBeforeUnmount } from 'vue';
import Terminal from './Terminal.vue';
import BlueScreen from './BlueScreen.vue';
import BootScreen from './BootScreen.vue';
import { BLUE_SCREEN_TIMEOUT, GLITCH_TEXT_TIMEOUT, TIME_BETWEEN_WORDS, TYPING_TIME } from './constants/timings.constant';

const textToDisplay = [
  'Hello visitor!',
  'I am Axel, a Web Developer.',
  'Welcome to my portfolio!',
];
const textIndex = ref<number>(0);
const displayedText = ref<string>('');
const textDisplayIntervalId = ref<number | undefined>(undefined);

const textDisplayIsOver = ref<boolean>(false);
const shouldShowBlueScreen = ref<boolean>(false);
const shouldShowBootScreen = ref<boolean>(false);
const shouldShowTerminal = ref<boolean>(false);


const displayNextText = () => {
  const text = textToDisplay[textIndex.value];
  const currentText = displayedText.value;

  if (currentText.length < text.length) {
    displayedText.value = text.slice(0, currentText.length + 1);
  } else {
    clearInterval(textDisplayIntervalId.value);
    if (textIndex.value === textToDisplay.length - 1) {
      textDisplayIsOver.value = true;
      clearInterval(underscoreIntervalId);
      setTimeout(() => {
        shouldShowBlueScreen.value = true;
        setTimeout(() => {
          shouldShowBlueScreen.value = false;
          shouldShowBootScreen.value = true;
        }, BLUE_SCREEN_TIMEOUT);
      }, GLITCH_TEXT_TIMEOUT);
      return
    }
    setTimeout(() => {
      textIndex.value = (textIndex.value + 1) % textToDisplay.length;
      displayedText.value = '';
      startDisplayingText();
    }, TIME_BETWEEN_WORDS);
  }
};

const startDisplayingText = () => {
  textDisplayIntervalId.value = setInterval(displayNextText, TYPING_TIME);
};

// Start displaying the first text
startDisplayingText();

const isUnderscoreVisible = ref<boolean>(true);
const underscoreIntervalId = setInterval(() => {
  isUnderscoreVisible.value = !isUnderscoreVisible.value;
}, 400);

onBeforeUnmount(() => {
  clearInterval(textDisplayIntervalId.value);
  clearInterval(underscoreIntervalId);
});
</script>

<template>
  <div v-if="shouldShowBootScreen">
    <BootScreen @logsEnded="shouldShowBlueScreen = false; shouldShowBootScreen = false; shouldShowTerminal = true"/>
  </div>
  <div v-else-if="shouldShowBlueScreen">
    <BlueScreen/>
  </div>
  <div v-else-if="shouldShowTerminal">
    <Terminal />
  </div>
  <div v-else>
    <div class="console-container">
      <span :class="{ glitch: textDisplayIsOver }">{{ displayedText }}</span>
      <span class="console-underscore" :class="{ hidden: isUnderscoreVisible, glitch: textDisplayIsOver  }">&#95;</span>
    </div>
  </div>
</template>

<style scoped>
@import url(https://fonts.googleapis.com/css?family=Khula:700);
@import url(https://fonts.googleapis.com/css?family=monospace:700);

.console-container {
  font-family: Khula;
  font-size: 3em;
  height: 200px;
  width: 80%;
  display: block;
  position: absolute;
  text-align: center;
  color: white;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  transition: all 0.5s ease;
}

.glitch {
  animation: glitch 0.2s steps(2) infinite;
}

@keyframes glitch {
  0% {
    text-shadow: 2px 2px red, -2px -2px blue;
  }
  25% {
    text-shadow: -2px -2px red, 2px 2px blue;
  }
  50% {
    transform: translate(2px, -2px);
  }
  75% {
    text-shadow: 2px 2px red, -2px -2px blue;
  }
  100% {
    transform: translate(-2px, 2px);
  }
}

.console-underscore.hidden {
  visibility: hidden;
}

@media (max-width: 768px) {
  .console-container {
    font-size: 2em;
    width: 80%;
    height: 200px;
  }
}

@media (max-width: 480px) {
  .console-container {
    font-size: 1.5em;
    width: 90%;
    height: 200px;
  }
}

.blue-screen {
  background-color: blue;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-size: 2rem;
}
</style>