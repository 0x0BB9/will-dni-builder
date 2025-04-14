<!-- eslint-disable style/brace-style -->
<!-- eslint-disable style/indent -->
<script setup lang="ts">
import { getFingerprint } from '@thumbmarkjs/thumbmarkjs'
import browser from 'browser-tool'
import { ElMessage } from 'element-plus'

import { onMounted, ref } from 'vue'

const input = ref('')
const fullDni = ref('')

defineProps<{ msg: string }>()

// 定义 loading 状态和 networkInfo 数据
const loading = ref(true)
const info = ref<any>(null)
const fingerPrint = ref<any>(null)
const fingerPrint2 = ref<any>(null)

// 页面初始化时加载 networkInfo
onMounted(async () => {
  // 测试用例
  console.log(validateSpanishDNI("12345678Z")); // 有效（假设Z对应余数）
  console.log(validateSpanishDNI("00000000T")); // 有效（全零但校验位正确）
  console.log(validateSpanishDNI("00000000A")); // 有效（全零但校验位正确）
})

function validateSpanishDNI(dni: string) {
  const pattern = /^(\d{8})([A-HJ-NP-TV-Z])$/i;
  const match = dni.toUpperCase().match(pattern);
  
  if (!match) return false;

  const digits = match[1];
  const letter = match[2];
  const remainder = parseInt(digits) % 23;

  return letter === DNI_LETTER_MAP.charAt(remainder);
}

// 字母映射表（官方规范）
const DNI_LETTER_MAP = 'TRWAGMYFPDXBNJZSQVHLCKE';

// 生成校验字母
function calculateCheckDigit(numbers: string) {
    const numericValue = parseInt(numbers, 10);
    if (isNaN(numericValue)) return '';
    return DNI_LETTER_MAP[numericValue % 23];
}

// 输入验证与处理
function validateInput(input: string) {
    return /^\d{8}$/.test(input); // 严格8位数字校验 
}

// 主生成函数
function generateDNI() {
    
    if (!validateInput(input.value)) {
      console
        return;
    }

    const checkLetter = calculateCheckDigit(input.value);
    const fullDNI = input.value + checkLetter;
    
    // 结果输出与复制功能
    fullDni.value = fullDNI;
}

// 生成8位随机数字（包含前导零）
function generateRandomNumber() {
    return Math.floor(Math.random() * 100000000)
           .toString()
           .padStart(8, '0'); // 强制补足8位 
}

// 主生成函数增强
function generateRandomDNI() {
    const randomNumber = generateRandomNumber();
    input.value = randomNumber; // 填充输入框
    generateDNI(); // 自动触发校验码生成
}

// 剪贴板操作（兼容现代浏览器） 
function copyToClipboard(text: string) {
    navigator.clipboard.writeText(text)
        .then(() => alert('已复制到剪贴板'))
        .catch(err => console.error('复制失败:', err));
}





</script>

<template>
  
  <div class="top-container">
    <div class="my-2">
    <el-input v-model="input" class="m-2" size="large" placeholder="输入8位数字" style="width: 200px" maxlength="8"/>
    <el-button size="large" @click="generateDNI()" type="primary">
      生成DNI
    </el-button>
    <el-button size="large" @click="generateRandomDNI()" type="primary">
      随机生成一个DNI
    </el-button>
    <h2>
        DNI: {{ fullDni }}
      </h2>
      <el-button size="large" @click="copyToClipboard(fullDni)" >
      复制DNI
    </el-button>
  </div>
  </div>
</template>

<style lang="scss" scoped>
.center-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  /* 使容器占满屏幕高度 */
  text-align: center;
  padding-bottom: 60px;
}

.el-table {
  margin-top: 20px;
}

/* Key 列样式 */
.key-column {
  font-weight: bold;
  /* 字体加粗 */
}
.container { max-width: 600px; margin: 2rem auto; padding: 20px; }
        input { padding: 8px; width: 200px; margin-right: 10px; }
        button { padding: 8px 15px; cursor: pointer; }
        .result { margin-top: 20px; padding: 10px; border: 1px solid #ddd; }
        .copy-btn { margin-left: 10px; }
</style>
