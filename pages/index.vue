<template>
  <div class="container">
    <div style="width: 60%;">
      <a-row>
        <h2>Training Data (x,y) pairs</h2>
        <a-col :span="12">X</a-col>
        <a-col :span="12">Y</a-col>
      </a-row>
      <a-row v-for="(item, idx) in xval" :key="idx" :gutters="[10, 10]">
        <a-col :span="12">
          <a-input v-model="xval[idx]" type="number" />
        </a-col>
        <a-col :span="12">
          <a-input v-model="yval[idx]" type="number" />
        </a-col>
      </a-row>
      <div style="margin-top: 20px;">
        <a-button block type="primary" @click="addItem">+</a-button>
        <a-button block @click="train">Train</a-button>
      </div>
      <a-row style="margin-top: 20px;">
        <a-col>
          <h2>Predicting</h2>
          <a-input v-model="toPredict" type="number" placeholder="Enter an integer number" />
          <div>{{ predictedVal }}</div>
          <a-button @click="predict" :disabled="!trained">Predict</a-button>
        </a-col>
      </a-row>
    </div>
  </div>
</template>

<script>
import * as tf from "@tensorflow/tfjs";

export default {
  data() {
    return {
      xval: [1, 2, 3, 4],
      yval: [1, 3, 5, 7],
      toPredict: "",
      predictedVal: "",
      trained: false,
      model: {}
    };
  },
  methods: {
    addItem() {
      this.xval.push(0);
      this.yval.push(0);
    },
    train() {
      const model = tf.sequential();
      model.add(tf.layers.dense({ units: 1, inputShape: [1] }));
      model.compile({
        loss: "meanSquaredError", //tf.losses.meanSquaredError,
        optimizer: "sgd" //tf.train.sgd
        // metrics: ["mse"]
      });

      const xs = tf.tensor2d(this.xval, [this.xval.length, 1]);
      const ys = tf.tensor2d(this.yval, [this.yval.length, 1]);

      model
        .fit(xs, ys, {
          epochs: 100,
          callbacks: {
            onEpochEnd: (epoch, log) =>
              console.log(`Epoch ${epoch}: loss=${log.loss}`, log)
          }
        })
        .then(() => {
          this.trained = true;
          this.predictedVal = "Ready for making prediction";
        });
      this.model = model;
    },
    async predict() {
      const input = parseInt(this.toPredict);
      const predicted = this.model.predict(tf.tensor2d([input], [1, 1]));
      // console.log(tf.tensor2d([parseInt(this.toPredict)], [1, 1]));
      console.log(predicted.arraySync(), predicted.toFloat());
      this.predictedVal = predicted.arraySync()[0];
    }
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
