# face-api-demo-vue

使用 `Vue` 框架搭建演示，`H5`、`Web`、`NodeJS` 实现人脸检测识别，基于 `TensorFlowJS` 实现的 `face-api.js` 人脸识别库。

识别模型文件放在 `public/models` 目录下，部署时请确保网络路径能访问到模型文件。

npm 主要安装识别库：

```shell
# 升级版作者库
npm i --save @vladmandic/face-api
# 原作者库
# npm i --save face-api.js
```

安装识别库库后在 `node_modules/@vladmandic/face-api/model` 可以拿到模型文件。

- [本人在线示例-aliyun](https://env-00jxgaqjulpu-static.normal.cloudstatic.cn/face-api-demo-vue/index.html#/)
- [本人在线示例-vercel](https://face-api-demo-vue-mu.vercel.app/)
- [本人仓库 browser 示例 0.13.8 含 dist/文档/模型/示例](https://github.com/TsMask/face-api-demo)
- [GitHub 原作者face-api 使用文档](https://justadudewhohacks.github.io/face-api.js/docs/globals.html)
- [GitHub 原作者识别库源码](https://github.com/justadudewhohacks/face-api.js)
- [GitHub 原作者在线示例](https://justadudewhohacks.github.io/face-api.js)
- [GitHub 升级版作者face-api 使用文档](https://vladmandic.github.io/face-api/typedoc/index.html)
- [GitHub 升级版作者识别库源码](https://github.com/vladmandic/face-api)
- [GitHub 升级版作者在线示例](https://vladmandic.github.io/face-api/demo/index.html)

## 其他

```js
console.log(faceapi.nets);
// const nets: {
//     ssdMobilenetv1: SsdMobilenetv1;
//     tinyFaceDetector: TinyFaceDetector;
//     tinyYolov2: TinyYolov2;
//     faceLandmark68Net: FaceLandmark68Net;
//     faceLandmark68TinyNet: FaceLandmark68TinyNet;
//     faceRecognitionNet: FaceRecognitionNet;
//     faceExpressionNet: FaceExpressionNet;
//     ageGenderNet: AgeGenderNet;
// }
//
// 放在public/models内
// 载入模型
// await faceapi.nets.ssdMobilenetv1.load("/models");
// 通过url地址
// await faceapi.nets.ssdMobilenetv1.loadFromUri("/models");
// 本地相对路径
// await faceapi.nets.ssdMobilenetv1.loadFromDisk("../../public/models");
```

### 侦测人脸

- detectAllFaces ：检测图像中的所有脸部
- detectSingleFace ：检测图像中具有最高置信度得分的脸部

默认情况下，`detectAllFaces` 和 `detectSingleFace` 使用 `SSD Mobilenet V1` 人脸检测器。

您可以通过传递相应的 options 对象来指定面部检测器：

```js
const detections1 = await faceapi.detectAllFaces(输入, 使用的模型参数)
const detections2 = await faceapi.detectSingleFace(输入, 使用的模型参数)
```

输入 ：使用 `HTMLImageElement | HTMLVideoElement | HTMLCanvasElement` 类型

使用的模型参数 ：配置识别度和修改人脸框，请看 `识别库说明` 或者 `查阅文档`。

### 识别库工具

```js
// 请求json格式文件数据 Any
const json = await faceapi.fetchJson("http://www.example.com/file/example.json");
// 请求视频文件 HTMLImageElement
const imageEl = await faceapi.fetchImage("http://www.example.com/file/example.jpg");
// 请求视频文件 HTMLVideoElement
const videoEl = await faceapi.fetchVideo("http://www.example.com/file/example.mp4");

// 从Blob/File对象创建HTMLImageElement
const imageEl = await faceapi.bufferToImage(fileBlob);

// 从图像或视频元素创建画布元素
const canvasEl = faceapi.createCanvasFromMedia(HTMLImageElement | HTMLVideoElement)
```

----

## 当前项目说明

示例参考 `face-api.js` 作者提供示例源码 `examples-browser`。

内含功能：

1. [脸标志检测](src/views/FaceLandmarkDetection.vue)
2. [人脸表情识别](src/views/FaceExpressionRecognition.vue)
3. [年龄和性别识别](src/views/AgeAndGenderRecognition.vue)
4. [人脸提取](src/views/FaceExtraction.vue)
5. [人脸识别](src/views/FaceRecognition.vue)
6. [人脸识别多图处理](src/views/FaceRecognitionMore.vue)
7. [BBT 人脸识别](src/views/BBTFaceRecognition.vue)
8. [BBT 图片相似度](src/views/BBTFaceSimilarity.vue)
9. [BBT 图片相似度多查找](src/views/BBTFaceSimilarityMore.vue)
10. [Video 人脸跟踪](src/views/VideoFaceDetector.vue)
11. [Video 人脸识别](src/views/VideoFaceRecognition.vue)
12. [WebRTC 人脸跟踪](src/views/WebRTCFaceDetector.vue)
13. [WebRTC 人脸识别](src/views/WebRTCFaceRecognition.vue)
14. [WebRTC 媒体流基础](src/views/WebRTCMediaStream.vue)

编译执行环境：

- NodeJS：`nodejs@18.16.0`
- Vite：`vite@5.2.8`
- Vue：`vue@3.4.21`
- VueRouter：`vue-router@4.3.0`
- face-api：`@vladmandic/face-api@1.7.13`

## 项目结构

```text
face-api-demo-vue
┌─public                  目录-静态资源
├─src
│ ├─assets                目录-编译资源
│ ├─components            目录-复用组件
│ ├─router                目录-页面路由配置
│ ├─utils                 目录-常用函数工具
│ ├─views                 目录-页面视图
│ ├─App.vue               文件-配置App全局
│ └─main.js               文件-Vue初始化
├─index.html              文件-页面容器
├─package-lock.json       文件-项目依赖版本锁
├─package.json            文件-项目信息
├─README.md               文件-项目说明
└─vite.config.js          文件-Vite配置
```

## Project setup / 项目依赖安装初始化

```bash
npm install
# 使用阿里源可以加速下载依赖库
npm install --registry https://registry.npmmirror.com
```

### Compiles and hot-reloads for development / 项目编译热更新用于开发调试

```bash
npm run dev
```

### Compiles and minifies for production / 项目编译打包用于部署

```bash
npm run build
```

### Customize configuration / 更多脚手架配置

See [Configuration Reference](https://vitejs.dev/config/).
