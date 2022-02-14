<script setup>
import { computed, nextTick, reactive, ref } from 'vue'
import html2canvas from 'html2canvas'

// 图片上传
const uploadRef = ref(null)
const imgSrc = ref('')
let imgType = ''
function handleChange(file) {
  imgSrc.value = URL.createObjectURL(file.raw)
  imgType = file.raw.type
}
function handleExceed(files) {
  URL.revokeObjectURL(imgSrc.value) // 释放图片对象
  // 替换上一个文件
  uploadRef.value.clearFiles()
  uploadRef.value.handleStart(files[0])
}

// 水印配置
const config = reactive({
  content: '水印文字',
  size: 36,
  color: '#666666',
  opacity: 0.8,
  rotate: -30,
  numX: 4,
  numY: 6,
  gapX: 40,
  gapY: 100,
  offsetX: -40,
  offsetY: 10,
})
const gapXVal = computed(() => `${config.gapX}px`)
const gapYVal = computed(() => `${config.gapY}px`)
const offsetXVal = computed(() => `${config.offsetX}px`)
const offsetYVal = computed(() => `${config.offsetY}px`)
const rotateVal = computed(() => `${config.rotate}deg`)
const sizeVal = computed(() => `${config.size}px`)

// 生成
const wrapperWidth = 600 // 预览宽度
const previewRef = ref(null)
const imgRef = ref(null)
const isGenerate = ref(false)
const watermarkScale = ref(1)
const previewStyle = computed(() => ({
  display: isGenerate.value ? 'inline-block' : 'block',
}))
const imgStyle = computed(() => ({
  width: isGenerate.value ? 'auto' : '100%',
}))
const isShowResult = ref(false)
const resultSrc = ref('')
async function generate() {
  if (!imgSrc.value) {
    ElMessage('请选择图片')
    return
  }
  isGenerate.value = true
  await nextTick()
  watermarkScale.value = imgRef.value.width / wrapperWidth
  await nextTick()
  try {
    const canvas = await html2canvas(previewRef.value)
    resultSrc.value = canvas.toDataURL(imgType)
    isShowResult.value = true
  } catch (error) {
    ElMessage.error('生成出错')
  }
  // 重置回预览状态
  watermarkScale.value = 1
  isGenerate.value = false
}
</script>

<template>
  <el-row :gutter="10">
    <el-col :span="8">
      <el-form :model="config" label-width="80px" class="form">
        <el-form-item label="图片">
          <el-upload
            ref="uploadRef"
            action=""
            :show-file-list="false"
            :limit="1"
            :auto-upload="false"
            :on-change="handleChange"
            :on-exceed="handleExceed"
          >
            <template #trigger>
              <el-button type="primary">选择文件</el-button>
            </template>
            <template #default></template>
          </el-upload>
        </el-form-item>
        <el-form-item label="内容">
          <el-input placeholder="水印内容" v-model="config.content" />
        </el-form-item>
        <el-form-item label="大小">
          <el-input-number :min="0" :step="2" v-model="config.size" />
        </el-form-item>
        <el-form-item label="颜色">
          <el-color-picker v-model="config.color" />
        </el-form-item>
        <el-form-item label="透明度">
          <el-slider
            :min="0"
            :max="1"
            :step="0.1"
            v-model="config.opacity"
          ></el-slider>
        </el-form-item>
        <el-form-item label="角度">
          <el-input-number
            :min="-90"
            :max="90"
            :step="10"
            v-model="config.rotate"
          />
        </el-form-item>
        <el-form-item label="X轴数量">
          <el-input-number :min="1" v-model="config.numX" />
        </el-form-item>
        <el-form-item label="Y轴数量">
          <el-input-number :min="1" v-model="config.numY" />
        </el-form-item>
        <el-form-item label="X轴间隔">
          <el-input-number :step="10" v-model="config.gapX" />
        </el-form-item>
        <el-form-item label="Y轴间隔">
          <el-input-number :step="10" v-model="config.gapY" />
        </el-form-item>
        <el-form-item label="X轴偏移">
          <el-input-number :step="10" v-model="config.offsetX" />
        </el-form-item>
        <el-form-item label="Y轴偏移">
          <el-input-number :step="10" v-model="config.offsetY" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="generate">生成</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <el-col :span="16">
      <div v-if="imgSrc" class="wrapper">
        <div ref="previewRef" class="preview" :style="previewStyle">
          <img
            ref="imgRef"
            :src="imgSrc"
            class="img"
            :style="imgStyle"
            @load="handleLoad"
          />
          <div class="watermark">
            <div v-for="y in config.numY" class="row">
              <div v-for="x in config.numX" class="col">
                {{ config.content }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </el-col>
  </el-row>

  <el-dialog title="右击图片保存" v-model="isShowResult">
    <img :src="resultSrc" class="result" />
  </el-dialog>
</template>

<style scoped>
.form {
  max-width: 400px;
}
.wrapper {
  overflow: hidden;
  width: v-bind(wrapperWidth + 'px');
}
.preview {
  position: relative;
}
.img {
  display: block;
}
.watermark {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
  font-size: v-bind(sizeVal);
  color: v-bind('config.color');
  opacity: v-bind('config.opacity');
  transform: scale(v-bind(watermarkScale))
    translate(v-bind(offsetXVal), v-bind(offsetYVal));
  transform-origin: 0 0;
}
.row {
  white-space: nowrap;
}
.col {
  display: inline-block;
  margin-right: v-bind(gapXVal);
  margin-bottom: v-bind(gapYVal);
  transform: rotate(v-bind(rotateVal));
}
.result {
  width: 100%;
}
</style>
