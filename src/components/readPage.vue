<template>
  <div>
    <p style="font-size: larger; font-weight: bold">{{ wordToShow }}</p>
    <input
      type="file"
      id="file"
      ref="myFiles"
      class="custom-file-input"
      accept="application/pdf"
      @change="previewFiles"
      multiple
    />
    <p>
      Mots par minutes : {{ wordPerMin }}
      <a href="#" @click="wordPerMin += 50">+</a>
      <a href="#" @click="wordPerMin -= 50">-</a>
    </p>
    <p class="playpause">
      <a v-if="pause"
        ><img
          @click="pause = !pause"
          style="width: 32px"
          src="../assets/img/play.png"
      /></a>
      <a v-else-if="!pause"
        ><img
          @click="pause = !pause"
          style="width: 32px"
          src="../assets/img/pause.png"
      /></a>
    </p>
    <p @click="(currentWord = 0), (wordToShow = ''), (pause = true)">reset</p>
  </div>
</template>
<script>
const pdfjsLib = require("pdfjs-dist");
pdfjsLib.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/2.15.349/pdf.worker.js`;
sessionStorage.clear();
export default {
  name: "readPage",
  props: {
    msg: String,
  },
  data() {
    return {
      wordsToRead: "",
      wordToShow: "",
      wordsArray: [],
      wordPerMin: 0,
      pause: true,
      currentWord: 0,
      files: [],
    };
  },
  methods: {
    textToArray(text) {
      text = text.replace("  ", " ");
      text = text.replace("\n", " ");
      this.wordsArray = text.split(" ");
    },
    sleep(milliseconds) {
      return new Promise((resolve) => setTimeout(resolve, milliseconds));
    },
    async playWordsLoop() {
      for (var i = this.currentWord; i < this.wordsArray.length; i++) {
        if (!this.pause) {
          this.wordToShow = this.wordsArray[i];
          this.currentWord = i;
          if (i == this.wordsArray.length - 1) {
            console.log("Text Terminé");
            this.wordsToRead = "";
          }
          await this.sleep((60 / this.wordPerMin) * 1000);
        }
      }
    },
    async previewFiles() {
      var file = event.target.files[0];

      //Step 2: Read the file using file reader
      var fileReader = new FileReader();

      fileReader.onload = function () {
        //Step 4:turn array buffer into typed array
        var typedarray = new Uint8Array(this.result);

        //Step 5:pdfjs should be able to read this
        const loadingTask = pdfjsLib.getDocument(typedarray);
        loadingTask.promise.then((pdf) => {
          let pageTexts = Array.from({ length: pdf.numPages }, async (v, i) => {
            return (await (await pdf.getPage(i + 1)).getTextContent()).items
              .map((token) => token.str)
              .join("");
          });

          return Promise.all(pageTexts).then((values) => {
            sessionStorage.clear();
            sessionStorage.setItem("text", values.join(" "));
            return values.join(" ");
          });
        });
      };

      //Step 3:Read the file as ArrayBuffer
      console.log("text bien chargé");
      fileReader.readAsArrayBuffer(file);
    },
  },
  watch: {
    pause: function (val) {
      if (!val && this.wordPerMin > 0) {
        if (!this.wordsToRead) {
          this.wordsToRead = sessionStorage.getItem("text");
          sessionStorage.clear();
        }
        this.textToArray(this.wordsToRead);
        this.playWordsLoop();
      }
    },
  },
};
</script>
<style scooped>
.playpause {
  display: flex;
  justify-content: center;
}
</style>
