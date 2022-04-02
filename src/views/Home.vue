<template>
  <div class="train">
    <template v-if="mode == 'train'">
      <h1>ถ่ายรูปตามอารมณ์ต่าง ๆ</h1>
    </template>
    <template v-else>
      <h1>มาลองเล่นกันเถอะ!</h1>
    </template>
    <select id="use_case" v-on:change="changeOption()">
      <option value="train">ถ่ายรูป</option>
      <option value="test">ลองเล่น</option>
    </select>
    <Camera></Camera>
    <template v-if="mode == 'train'">
      <select id="emotion_options">
        <template v-for="(emotion, index) in emotions">
          <option :key="index" :value="index">{{ emotion }}</option>
        </template>
      </select>
      <button v-on:click="trainModel()">ถ่าย แชะ!</button>
    </template>
    <template v-else>
      <button v-on:click="getEmotion()">ตรวจจับอารมณ์</button>
      <p>
        <!-- <h1>{{ detected_e }}</h1> -->

        <button v-on:click="angry()">โกรธ</button>
        <button v-on:click="neutral()">เฉยๆ</button>
        <button v-on:click="happy()">มีความสุข</button>
        <button v-on:click="sad()">เศร้า</button>
        <button v-on:click="surprise()">ประหลาดใจ</button>
      </p>
      <h1>{{ score }}</h1>
    </template>
  </div>
</template>

<script>
// @ is an alias to /src
import Camera from "@/components/Camera.vue";
import * as tf from "@tensorflow/tfjs";
import * as mobilenetModule from "@tensorflow-models/mobilenet";
import * as knnClassifier from "@tensorflow-models/knn-classifier";
import axios from "axios";

export default {
  name: "Home",
  components: {
    Camera,
  },
  data: function () {
    return {
      emotions: ["โกรธ", "เฉยๆ", "มีความสุข", "เศร้า", "ประหลาดใจ"],
      classifier: null,
      mobilenet: null,
      class: null,
      detected_e: null,
      mode: "train",
      score: 0,
    };
  },
  mounted: function () {
    this.init();
  },
  methods: {
    async init() {
      // load the load mobilenet and create a KnnClassifier
      this.classifier = knnClassifier.create();
      this.mobilenet = await mobilenetModule.load();
    },
    trainModel() {
      let selected = document.getElementById("emotion_options");
      this.class = selected.options[selected.selectedIndex].value;
      this.addExample();
    },
    addExample() {
      const img = tf.fromPixels(this.$children[0].webcam.webcamElement);
      const logits = this.mobilenet.infer(img, "conv_preds");
      this.classifier.addExample(logits, parseInt(this.class));
    },
    async getEmotion() {
      const img = tf.fromPixels(this.$children[0].webcam.webcamElement);
      const logits = this.mobilenet.infer(img, "conv_preds");
      const pred = await this.classifier.predictClass(logits);
      console.log(pred);
      this.detected_e = this.emotions[pred.classIndex];
      this.registerEmotion();
    },
    angry() {
      if (this.detected_e == "โกรธ") {
        this.score = this.score + 1;
        console.log(this.score);
      }
    },
    neutral() {
      if (this.detected_e == "เฉยๆ") {
        this.score = this.score + 1;
        console.log(this.score);
      }
    },
    happy() {
      if (this.detected_e == "มีความสุข") {
        this.score = this.score + 1;
        console.log(this.score);
      }
    },
    sad() {
      if (this.detected_e == "เศร้า") {
        this.score = this.score + 1;
        console.log(this.score);
      }
    },
    surprise() {
      if (this.detected_e == "ประหลาดใจ") {
        this.score = this.score + 1;
        console.log(this.score);
      }
    },
    changeOption() {
      const selected = document.getElementById("use_case");
      this.mode = selected.options[selected.selectedIndex].value;
    },
    registerEmotion() {
      axios
        .post("http://localhost:3128/callback", {
          emotion: this.detected_e,
        })
        .then(() => {
          alert("Thanks for letting us know how you feel");
        });
    },
  },
};
</script>
