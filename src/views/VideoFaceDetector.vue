<script setup>
import * as faceapi from "@vladmandic/face-api";
import { onMounted, onUnmounted, reactive } from "vue";

/**属性状态 */
const state = reactive({
  /**初始化模型加载 */
  netsLoadModel: true,
  /**算法模型 */
  netsType: "ssdMobilenetv1",
  /**模型参数 */
  netsOptions: {
    ssdMobilenetv1: undefined,
    tinyFaceDetector: undefined,
  },
  /**检测人脸 多或单 */
  detectFace: "detectAllFaces",
  /**面框分值, 面部轮廓, 面部表情, 年龄性别 */
  draws: ["box", "landmark", "expression", "ageGender"],
  /**视频元素 */
  videoEl: null,
  /**画布图层元素 */
  canvasEl: null,
  /**绘制定时器 */
  timer: 0,
});

/**初始化模型加载 */
async function fnLoadModel() {
  // 模型文件访问路径
  const modelsPath = `/models`;
  // 面部轮廓模型
  await faceapi.nets.faceLandmark68Net.load(modelsPath);
  // 面部表情模型
  await faceapi.nets.faceExpressionNet.load(modelsPath);
  // 年龄性别模型
  await faceapi.nets.ageGenderNet.load(modelsPath);

  // 模型参数-ssdMobilenetv1
  await faceapi.nets.ssdMobilenetv1.load(modelsPath);
  state.netsOptions.ssdMobilenetv1 = new faceapi.SsdMobilenetv1Options({
    minConfidence: 0.5, // 0 ~ 1
    maxResults: 50, // 0 ~ 100
  });
  // 模型参数-tinyFaceDetector
  await faceapi.nets.tinyFaceDetector.load(modelsPath);
  state.netsOptions.tinyFaceDetector = new faceapi.TinyFaceDetectorOptions({
    inputSize: 416, // 160 224 320 416 512 608
    scoreThreshold: 0.5, // 0 ~ 1
  });

  // 输出库版本
  console.log(
    `FaceAPI Version: ${
      faceapi?.version || "(not loaded)"
    } \nTensorFlow/JS Version: ${
      faceapi.tf?.version_core || "(not loaded)"
    } \nBackend: ${
      faceapi.tf?.getBackend() || "(not loaded)"
    } \nModels loaded: ${faceapi.tf.engine().state.numTensors} tensors`
  );

  // 节点元素
  state.videoEl = document.getElementById("page_draw-video");
  state.canvasEl = document.getElementById("page_draw-video-canvas");

  // 关闭模型加载
  state.netsLoadModel = false;
}

/**根据模型参数识别绘制 */
async function fnRedraw() {
  if (!state.videoEl || !state.canvasEl) return;
  console.log("Run Redraw");

  // 暂停视频时清除定时
  if (state.videoEl.paused) {
    clearTimeout(state.timer);
    state.timer = 0;
    return;
  }

  // 识别绘制人脸信息
  const detect = await faceapi[state.detectFace](
    state.videoEl,
    state.netsOptions[state.netsType]
  )
    // 需引入面部轮廓模型
    .withFaceLandmarks()
    // 需引入面部表情模型
    .withFaceExpressions()
    // 需引入年龄性别模型
    .withAgeAndGender();

  // 无识别数据时，清除定时重新再次识别
  if (!detect) {
    clearTimeout(state.timer);
    state.timer = 0;
    fnRedraw();
    return;
  }

  // 匹配元素大小
  const dims = faceapi.matchDimensions(state.canvasEl, state.videoEl, true);
  const result = faceapi.resizeResults(detect, dims);

  // 面框分值
  if (state.draws.includes("box")) {
    faceapi.draw.drawDetections(state.canvasEl, result);
  }
  // 面部轮廓
  if (state.draws.includes("landmark")) {
    // 需引入面部轮廓模型
    faceapi.draw.drawFaceLandmarks(state.canvasEl, result);
  }
  // 面部表情
  if (state.draws.includes("expression")) {
    // 需引入面部表情模型
    faceapi.draw.drawFaceExpressions(state.canvasEl, result, 0.05);
  }
  // 年龄性别
  if (state.draws.includes("ageGender")) {
    // 需引入年龄性别模型模型
    const drawItem = (item) => {
      const { age, gender, genderProbability } = item;
      new faceapi.draw.DrawTextField(
        [
          `${Math.round(age)} Age`,
          `${gender} (${Math.round(genderProbability)})`,
        ],
        item.detection.box.topRight
      ).draw(state.canvasEl);
    };
    // 多结果
    if (Array.isArray(result)) {
      result.forEach((item) => drawItem(item));
    } else {
      drawItem(result);
    }
  }

  // 定时器句柄
  state.timer = setTimeout(() => fnRedraw(), 0);
}

/**
 * 视频暂停播放
 *
 * 播放视频，开始识别绘制
 *
 * 暂停视频，不清除画布
 */
function fnVidelPaused() {
  if (state.videoEl.paused) {
    state.videoEl.play();
    setTimeout(() => fnRedraw(), 300);
  } else {
    state.videoEl.pause();
  }
}

/**更换视频 */
function fnChange(e) {
  if (!state.videoEl || !state.canvasEl) return;
  if (!e.target || !e.target.files.length) return;
  // 将文件显示为视频并识别
  state.videoEl.pause();
  state.videoEl.src = URL.createObjectURL(e.target.files[0]);
  clearTimeout(state.timer);
  state.timer = 0;

  setTimeout(() => {
    // 清空画布
    state.canvasEl
      .getContext("2d")
      .clearRect(0, 0, state.canvasEl.width, state.canvasEl.height);
  }, 500);
}

onMounted(() => {
  fnLoadModel();
});

onUnmounted(() => {
  clearTimeout(state.timer);
  state.timer = 0;
});
</script>

<template>
  <div class="page">
    <div class="page_option">
      <div>
        <label>更换视频：</label>
        <input
          type="file"
          accept="video/mp4, video/ogg, video/webm"
          @change="fnChange($event)"
        />
      </div>
      <div>
        <label>视频媒体：</label>
        <button @click="fnVidelPaused()">播放/暂停</button>
      </div>
      <div>
        <label>面框分值：</label>
        <input type="checkbox" value="box" v-model="state.draws" />
      </div>
      <div>
        <label>面部轮廓：</label>
        <input type="checkbox" value="landmark" v-model="state.draws" />
      </div>
      <div>
        <label>面部表情：</label>
        <input type="checkbox" value="expression" v-model="state.draws" />
      </div>
      <div>
        <label>年龄性别：</label>
        <input type="checkbox" value="ageGender" v-model="state.draws" />
      </div>
      <div>
        <label>算法模型：</label>
        <select v-model="state.netsType">
          <option value="ssdMobilenetv1">SSD Mobilenet V1</option>
          <option value="tinyFaceDetector">Tiny Face Detector</option>
        </select>
      </div>
      <div>
        <label>检测人脸：</label>
        <select v-model="state.detectFace">
          <option value="detectSingleFace">Detect Single Face</option>
          <option value="detectAllFaces">Detect All Faces</option>
        </select>
      </div>
    </div>

    <div class="page_load" v-show="state.netsLoadModel">Load Model...</div>
    <div class="page_draw" v-show="!state.netsLoadModel">
      <video
        id="page_draw-video"
        poster="/images/720x480.png"
        src="/videos/test.mp4"
        muted
        playsinline
      ></video>
      <canvas id="page_draw-video-canvas"></canvas>
    </div>
  </div>
</template>

<style scoped></style>
