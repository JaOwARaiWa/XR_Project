<template>
  <div class="train">
    <header>
      <template v-if="mode == 'train'">
        <h1>ถ่ายรูปตามอารมณ์ต่าง ๆ</h1>
      </template>
      <template v-else>
        <h1>มาลองเล่นกันเถอะ!</h1>
      </template>

      <div class="slide">
        <select id="use_case" v-on:change="changeOption()">
          <option value="train">ถ่ายรูป</option>
          <option :disabled="disabledButton" value="test">ลองเล่น</option>
        </select>
      </div>
    </header>

    <Tutorial v-if="isModalVisible == true" />

    <nav>
      <Camera></Camera>
    </nav>

    <article>
      <template v-if="mode == 'train'">
        <div class="slide">
          <select class="select-two" id="emotion_options">
            <template v-for="(emotion, index) in emotions">
              <option :key="index" :value="index">{{ emotion }}</option>
            </template>
          </select>
        </div>

        <button class="button-two" v-on:click="trainModel()">ถ่าย แชะ!</button>
      </template>

      <template v-else>
        <button class="button-three" v-on:click="getEmotion()">
          ตรวจจับอารมณ์
        </button>
        <h3>กรุณาเลือกคำตอบ</h3>
        <p>
          <!-- <h1>{{ detected_e }}</h1> -->
          <button
            class="button-one"
            :disabled="isDisabled"
            v-on:click="answer_emotion('โกรธ')"
          >
            โกรธ
          </button>
          <button
            class="button-one"
            :disabled="isDisabled"
            v-on:click="answer_emotion('เฉยๆ')"
          >
            เฉยๆ
          </button>
          <button
            class="button-one"
            :disabled="isDisabled"
            v-on:click="answer_emotion('มีความสุข')"
          >
            มีความสุข
          </button>
          <button class="button-one" 
          :disabled="isDisabled" 
          v-on:click="answer_emotion('เศร้า')">
            เศร้า
          </button>
          <button
            class="button-one"
            :disabled="isDisabled"
            v-on:click="answer_emotion('ประหลาดใจ')"
          >
            ประหลาดใจ
          </button>
        </p>
        <h3>คะแนน : {{ score }}</h3>
      </template>
    </article>
  </div>
</template>

<script>
// @ is an alias to /src
import Camera from "@/components/Camera.vue";
import * as tf from "@tensorflow/tfjs";
import * as mobilenetModule from "@tensorflow-models/mobilenet";
import * as knnClassifier from "@tensorflow-models/knn-classifier";
import axios from "axios";
import Tutorial from '@/components/Tutorial.vue';

export default {
  name: "Home",
  components: {
    Camera,
    Tutorial
  },
  data: function () {
    return {
      disabledButton: true,
      saveEmotionList: [],
      isDisabled: true,
      emotions: ["โกรธ", "เฉยๆ", "มีความสุข", "เศร้า", "ประหลาดใจ"],
      classifier: null,
      mobilenet: null,
      class: null,
      detected_e: null,
      mode: "train",
      score: 0,
      isModalVisible: true,
    };
  },
  mounted() {
    const thisInstance = this
    this.$root.$on('closeModal', function() {
      thisInstance.closeModal()
    });

    this.init();
  },
  methods: {
    async init() {
      // load the load mobilenet and create a KnnClassifier
      this.classifier = knnClassifier.create();
      this.mobilenet = await mobilenetModule.load();
    },
    closeModal() {
      this.isModalVisible = false;
    },
    trainModel() {
      let selected = document.getElementById("emotion_options");
      this.class = selected.options[selected.selectedIndex].value;
      this.addExample();
      // this.saveEmotionList.push(this.class);
      this.$swal(
        "บันทึกอารมณ์สำเร็จ",
        "สามารถเริ่มเล่นเกมได้แล้ว",
        "success"
      );
      this.disabledButton = false;
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
      this.isDisabled = false;
      this.$swal("ตรวจจับอารมณ์เรียบร้อย", "", "success");
    },
    answer_emotion(emote) {
      if (this.detected_e == emote) {
        this.score = this.score + 1;
        console.log(this.score);
        this.isDisabled = true;
        var audio = new Audio(require('@/assets/CorrectSound.mp3'));
        audio.play();
        this.$swal("ถูกต้อง", "เอาไปเลย 1 คะแนน", "success");
      } else {
        var audio = new Audio(require('@/assets/IncorrectSound.mp3'));
        audio.play();
        this.$swal("ยังไม่ถูกนะ", "ลองใหม่น้า", "error");
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
<style>
h1 {
  font-size: 55px;
}
h3 {
  color: white;
  font-size: 25px;
  margin-bottom: 30px;
}
button {
  padding: 0;
  border: none;
  font: inherit;
  color: rgb(255, 255, 255);
  background-color: transparent;
  cursor: pointer;
  
}
button:focus {
  outline: 0;
}
button:disabled {
  background-color: rgb(164, 164, 164);
  color: rgb(255, 255, 255);
  cursor: not-allowed;
}

.fas,
.far {
  padding-right: 5px;
}
.button-two {
  padding: 10px 15px;
  background-color: rgb(45, 153, 189);
  border-radius: 10px;
  width: 200px;
  height: 60px;
  font-size: 1.8em;
}
.button-two:hover {
  box-shadow: 0 3px 6px black;
}

.button-two:active {
  transform: translateY(2px);
}

.button-one {
  /* padding: 10px 15px; */
  margin: 10px;
  padding-bottom: 45px;
  background-color: rgb(189, 45, 151);
  border-radius: 10px;
  width: 170px;
  height: 40px;
  font-size: 1.8em;
}
.button-one:hover {
  box-shadow: 0 3px 6px black;
}

.button-one:active {
  transform: translateY(2px);
}

.button-three {
  margin-top: 45px;
  margin-bottom: 10px;
  padding: 10px 10px;
  background-color: rgb(45, 153, 189);
  border-radius: 10px;
  width: 250px;
  height: 60px;
  font-size: 1.8em;
}
.button-three:hover {
  box-shadow: 0 3px 6px black;
}

.button-three:active {
  transform: translateY(2px);
}

/* dropdown select */
option {
  font-family: 'Mitr';
  color: #000000;
  text-align: center;
}

select {
  font-family: 'Mitr';
  /* padding: 10px; */
  background: #ffffff;
  color: rgb(0, 0, 0);
  font-size: 1.8em;
  /* margin: 10px; */
  border-radius: 10px;
  width: 200px;
  height: 60px;
}

.select-two {
  margin: 100px;
}

.slide {
  margin: 20px;
}

nav {
  /* background-color: #000000; */
  background-color: #646490;

  background-size: 20px 20px;
  background-image: repeating-linear-gradient(
    to right,
    #ffd7f6,
    #ffe6f9 1px,
    #646490 1px,
    #646490
  );
  padding-top: 25px;
  text-align: center;
  float: left;
  width: 50%;
  height: 425px;
}

article {
  background-color: #e5e5f7;
  background-image: radial-gradient(#e7ffd1 0.5px, #1a1a46 0.5px);
  background-size: 10px 10px;
  padding: 0px;
  text-align: center;
  float: left;
  width: 50%;

  height: 450px;
}

header {
  /* background-color: rgb(167, 167, 167); */
  background-color: #e5e5f7;
  background-image: radial-gradient(#444cf7 0.5px, #d0d0fa 0.5px);
  background-size: 10px 10px;
  padding: 20px;
  text-align: center;
  /* font-size: 35px; */
  color: rgb(0, 0, 0);
}

footer {
  background: #3e3e3e;
}
</style>
