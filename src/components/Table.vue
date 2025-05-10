<!-- eslint-disable no-console -->
<!-- eslint-disable style/no-trailing-spaces -->
<!-- eslint-disable no-alert -->
<!-- eslint-disable vue/block-tag-newline -->
<!-- eslint-disable style/no-multiple-empty-lines -->
<!-- eslint-disable style/brace-style -->
<!-- eslint-disable style/indent -->
<script setup lang="ts">
import { getFingerprint } from '@thumbmarkjs/thumbmarkjs'
import browser from 'browser-tool'
import { ElMessage } from 'element-plus'

import { onMounted, ref } from 'vue'

defineProps<{ msg: string }>()
const input = ref('')
const fullDni = ref('')

const inputNie = ref('')
const fullNie = ref('')

// 定义 loading 状态和 networkInfo 数据
const loading = ref(true)
const info = ref<any>(null)
const fingerPrint = ref<any>(null)
const fingerPrint2 = ref<any>(null)

// 页面初始化时加载 networkInfo
onMounted(async () => {
  // 测试用例
  console.log(validateSpanishDNI('12345678Z')) // 有效（假设Z对应余数）
  console.log(validateSpanishDNI('00000000T')) // 有效（全零但校验位正确）
  console.log(validateSpanishDNI('00000000A')) // 有效（全零但校验位正确）
})

function validateSpanishDNI(dni: string) {
  const pattern = /^(\d{8})([A-HJ-NP-TV-Z])$/i
  const match = dni.toUpperCase().match(pattern)

  if (!match)
    return false

  const digits = match[1]
  const letter = match[2]
  const remainder = Number.parseInt(digits) % 23

  return letter === DNI_LETTER_MAP.charAt(remainder)
}

// 字母映射表（官方规范）
const DNI_LETTER_MAP = 'TRWAGMYFPDXBNJZSQVHLCKE'

// 生成校验字母
function calculateCheckDigit(numbers: string) {
  const numericValue = Number.parseInt(numbers, 10)
  if (isNaN(numericValue))
    return ''
  return DNI_LETTER_MAP[numericValue % 23]
}

// 输入验证与处理
function validateInput(input: string) {
  return /^\d{8}$/.test(input) // 严格8位数字校验 
}

// 主生成函数
function generateDNI() {
  if (!validateInput(input.value)) {
    ElMessage.error('请输入8位数字')
    return
  }

  const checkLetter = calculateCheckDigit(input.value)
  const fullDNI = input.value + checkLetter

  // 结果输出与复制功能
  fullDni.value = fullDNI
}


// 主生成函数增强
function generateRandomDNI() {
  const randomNumber = Math.floor(Math.random() * 100000000)
    .toString()
    .padStart(8, '0') // 强制补足8位 
  input.value = randomNumber // 填充输入框
  generateDNI() // 自动触发校验码生成
}

// 主生成函数增强
function generateRandomNIE() {
  const randomNumber = Math.floor(Math.random() * 10000000)
    .toString()
    .padStart(7, '0') // 强制补足8位 
  inputNie.value = randomNumber // 填充输入框
  generateNIE() // 自动触发校验码生成
}

// 剪贴板操作（兼容现代浏览器） 
function copyToClipboard(text: string) {
  navigator.clipboard.writeText(text)
    .then(() => alert('已复制到剪贴板'))
    .catch(err => console.error('复制失败:', err))
}
// 生成随机 NIE
function generateNIE() {
  // if (!(/^\d{7}$/.test(inputNie.value))) {
  //   return
  // }

  // 1. 生成首字母
  let prefix = ['X', 'Y', 'Z'][Math.floor(Math.random() * 3)]

  if (inputNie.value.length === 8) {
    prefix = inputNie.value[0]
    inputNie.value = inputNie.value.slice(1) // 去掉首字母
  } else if (inputNie.value.length !== 7) {
    ElMessage.error('请输入7位数字')
    return
  }

  // 2. 生成7位数字（含前导零）
  const numbers = inputNie.value

  // 3. 计算校验位
  const prefixNumber = prefix === 'X' ? 0 : (prefix === 'Y' ? 1 : 2)
  const fullNumber = Number.parseInt(prefixNumber + numbers)
  const remainder = fullNumber % 23
  console.log(`前缀数字: ${prefixNumber}, 余数: ${remainder}`)

  const controlLetter = DNI_LETTER_MAP[remainder]

  // 4. 组合最终结果
  const nie = `${prefix}${numbers}${controlLetter}`

  // 结果输出与复制功能
  fullNie.value = nie
}

const multipleDni = ref<string[]>([]) // 存储 100 个随机生成的 DNI

function generateMultipleDNI() {
  const dniList: string[] = []
  for (let i = 0; i < 100; i++) {
    const randomNumber = Math.floor(Math.random() * 100000000)
      .toString()
      .padStart(8, '0') // 强制补足8位
    const checkLetter = calculateCheckDigit(randomNumber)
    dniList.push(randomNumber + checkLetter)
  }
  multipleDni.value = dniList // 更新结果
}

const fullIban = ref('') // 存储生成的 IBAN

function generateRandomIBAN() {
  const countryCode = 'ES' // 西班牙国家代码
  const bankCode = Math.floor(Math.random() * 10000).toString().padStart(4, '0') // 银行代码
  const branchCode = Math.floor(Math.random() * 10000).toString().padStart(4, '0') // 分行代码
  const accountNumber = Math.floor(Math.random() * 10000000000).toString().padStart(10, '0') // 账户号码

  // 计算校验位
  const checkDigitsBase = `${bankCode}${branchCode}${accountNumber}${countryCode}00`
  const numericIBAN = checkDigitsBase
    .replace(/[A-Z]/g, char => (char.charCodeAt(0) - 55).toString()) // 替换字母为数字
  const checkDigits = (98n - BigInt(numericIBAN) % 97n).toString().padStart(2, '0') // 计算校验位

  // 组合最终 IBAN
  fullIban.value = `${countryCode}${checkDigits}${bankCode}${branchCode}${accountNumber}`
}

</script>

<template>
  <div class="top-container">
    <div class="center-bg my-2">
      <div class="center-h">
        <el-input v-model="input" class="m-2" size="large" placeholder="输入8位数字" style="width: 200px" maxlength="8" />
        <el-button size="large" type="primary" @click="generateDNI()">
          生成DNI
        </el-button>
        <el-button size="large" type="primary" @click="generateRandomDNI()">
          随机生成一个DNI
        </el-button>
      </div>

      <div class="center-h">
        <h2 m-2>
          DNI: {{ fullDni }}
        </h2>
        <el-button v-if="fullDni" size="large" @click="copyToClipboard(fullDni)">
          复制
        </el-button>
      </div>
    </div>

    <div class="center-bg my-2">
      <el-button size="large" type="primary" @click="generateMultipleDNI()">
        随机生成 100 个 DNI
      </el-button>
      <div v-if="multipleDni.length" class="result-list">
        <h3>随机生成的 100 个 DNI:</h3>
        <ul>
          <li v-for="(dni, index) in multipleDni" :key="index">
            {{ dni }}
          </li>
        </ul>
      </div>
    </div>

    <div class="center-bg">
      <div class="center-h">
        <el-input v-model="inputNie" class="m-2" size="large" placeholder="输入7位数字" style="width: 200px" maxlength="8" />
        <el-button size="large" type="primary" @click="generateNIE()">
          生成NIE
        </el-button>
        <el-button size="large" type="primary" @click="generateRandomNIE()">
          随机生成一个NIE
        </el-button>
      </div>

      <div class="center-h">
        <h2 m-2>
          NIE: {{ fullNie }}
        </h2>
        <el-button v-if="fullNie" size="large" @click="copyToClipboard(fullNie)">
          复制
        </el-button>
      </div>
    </div>

    <div class="center-bg my-2">
      <div class="center-h">
        <el-button size="large" type="primary" @click="generateRandomIBAN()">
          随机生成一个西班牙 IBAN
        </el-button>
      </div>
      <div v-if="fullIban" class="center-h">
        <h2 m-2>
          IBAN: {{ fullIban }}
        </h2>
        <el-button size="large" @click="copyToClipboard(fullIban)">
          复制
        </el-button>
      </div>
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

.center-bg {
  background-color: #f5f5f5;
  /* 添加背景颜色 */
  padding: 20px;
  /* 内边距 */
  border-radius: 8px;
  /* 圆角 */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  /* 水平居中 */
  max-width: 600px;
  /* 限制宽度 */
  margin-top: 20px;

  .center-h {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 20px;
  }
}

.el-table {
  margin-top: 20px;
}

/* Key 列样式 */
.key-column {
  font-weight: bold;
  /* 字体加粗 */
}

.container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 20px;
}

input {
  padding: 8px;
  width: 200px;
  margin-right: 10px;
}

button {
  padding: 8px 15px;
  cursor: pointer;
}

.result {
  margin-top: 20px;
  padding: 10px;
  border: 1px solid #ddd;
}

.copy-btn {
  margin-left: 10px;
}
</style>
