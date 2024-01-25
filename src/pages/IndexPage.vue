<script setup lang="ts">
import { createWorker } from "tesseract.js";

const imgPreview = ref('');
const handleChangeImg = (e: Event) => {
  const file = e?.target?.files?.[0];
  // 只选择图片文件
  if (!file.type.match('image.*')) {
    return false;
  }

  const reader = new FileReader();

  reader.readAsDataURL(file); // 读取文件

  // 渲染文件
  reader.onload = function(arg) {
    imgPreview.value = arg.target?.result as string ?? '';
    translate(file);
  }
}

const handleCopy = () => {
  if (!output.value) {
    alert('未生成内容');
    return;
  }
  navigator.clipboard.writeText(output.value);
  alert('复制成功');
}

// 输出文案
const output = ref('');
const loggerData = ref<Tesseract.LoggerMessage>({
  jobId: "",
  progress: 0,
  status: "unload",
  userJobId: "",
  workerId: ""
});
const progress = computed(() => {
  const num = loggerData.value?.progress.toFixed(2) ?? 0;
  return Number(num) * 100 + '%';
});
const worker = ref<Tesseract.Worker>();

async function initWorker() {
  worker.value = await createWorker('eng+chi_sim', 1, {
    workerPath: '/tesseract/worker/worker.min.js',
    // 下载其他语言训练文件 https://tessdata.projectnaptha.com/[lang].traineddata.gz
    langPath: '/tesseract/lang',
    corePath: '/tesseract/core',
    logger: m => loggerData.value = m, // Add logger here
  });
  
}

initWorker();

async function translate(file: File) {
  output.value = '';
  const { data: { text } } = await worker.value!.recognize(file);
  output.value = text;
}
</script>

<template>
  <div class="relative py-8">
    <div
      class="absolute inset-0 bg-[url(/img/grid.svg)] bg-top [mask-image:linear-gradient(180deg,white,rgba(255,255,255,0))]"
    ></div>
    <div
      class="p-6 container relative mx-auto bg-white shadow-xl shadow-slate-700/10 ring-1 ring-gray-900/5"
    >
      <main>
        <div class="text-center">
          <button class="mr-2 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="handleCopy">复制</button>
          <input type="file" name="selectImg" id="selectImg" @change="handleChangeImg" accept="image/*">
        </div>
        <div class="mt-2 flex justify-center">
          <img v-if="imgPreview" :src="imgPreview" width="500" alt="preview">
          <div v-else class="pre-img border border-black border-solid flex justify-center items-center mt-2">
            <span>
              图片预览
            </span>
          </div>
        </div>

        <div class="my-2">
          <span>输出文案如下：</span>

          <pre v-if="output" class=" overflow-auto">{{ output }}</pre>
          <div v-else>
            OCR组件状态 {{ loggerData?.status }}
            {{ progress }}
          </div>
        </div>
      </main>
    </div>
  </div>
</template>
<style>
.page {
  min-width: 1366px;
}
.pre-img {
  min-width: 350px;
  min-height: 350px;
}
</style>