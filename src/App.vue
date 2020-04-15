<template>
  <div>
    <app-header />
    <div class="app-container">
      <select v-model="selectedVoice">
        <option v-for="voice in voiceList" :key="voice.voiceURI">
          {{ voice.name }}
        </option>
      </select>
      <br />
      <div class="slidercontainer">
        <input
          v-model="speed"
          ref="speed"
          class="slider"
          type="range"
          min="0.5"
          max="3"
        />
        {{ speed }}
      </div>
      <span></span>
      <br />
      <textarea cols="30" rows="10" v-model="textToSpeech"></textarea>
      <div class="previewContainer">
        <span
          :ref="`spoken_word_${i}`"
          v-for="(word, i) in textToSpeech.split(' ')"
          :key="i"
        >
          {{ word }}
        </span>
      </div>
      <br />
      <button class="btn red" type="button" @click="speak">
        <span>Söyle Bakalım Şekerim!</span>
      </button>
    </div>
  </div>
</template>

<script>
import appHeader from "@/components/appHeader";
export default {
  name: "App",
  components: {
    appHeader
  },
  mounted() {
    this.$refs.speed.step = 0.5;
  },
  created() {
    // this.$refs.speed.step = 0.5;
    this.getVoices().then(voices => {
      this.voiceList = voices;
      // voiceURI, name == Yelda..
      // console.log("this.voiceList", this.voiceList);
      this.selectedVoice = "Yelda";

      // document.querySelector(".slider") = this.$refs.speed
      // this.$refs.speed.step = 0.5;
    });
  },
  data() {
    return {
      tts: window.speechSynthesis,
      voiceList: null,
      selectedVoice: null,
      textToSpeech: "Bir ihtimal daha var... O da ölmek mi dersin?",
      speed: 1
    };
  },
  methods: {
    getVoices() {
      let intervalID;
      return new Promise((resolve, reject) => {
        intervalID = setInterval(() => {
          if (this.tts.getVoices().length > 0) {
            resolve(this.tts.getVoices());
            clearInterval(intervalID);
          }
        }, 10);
      });
    },
    speak() {
      let toSpeak = new SpeechSynthesisUtterance(this.textToSpeech);
      toSpeak.voice =
        this.voiceList.find(v => v.name == this.selectedVoice) || null;
      toSpeak.rate = this.speed;

      toSpeak.onboundary = this.onBoundary;

      this.tts.speak(toSpeak);
    },
    onBoundary(event) {
      const spokenWord = event.utterance.text.substr(
        event.charIndex,
        event.charLength
      );
      const wordIndex = this.textToSpeech
        .split(" ")
        .findIndex(w => w == spokenWord);

      this.$refs[`spoken_word_${wordIndex}`][0].classList.add("spoken_word");

      console.log(
        "spoken",
        spokenWord,
        wordIndex,
        this.$refs[`spoken_word_${wordIndex}`]
      );
    }
  }
};
</script>
