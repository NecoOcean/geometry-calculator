<script setup>
import { ref, computed } from 'vue'

// 形状和尺寸参数
const shape = ref('frustum') // 'frustum' 或 'cone'
const topRadius = ref('')
const bottomRadius = ref('')
const height = ref('')
const thickness = ref('') // 板材厚度

// 单位选择
const lengthUnits = [
  { value: 0.01, label: 'mm', name: '毫米' },
  { value: 0.1, label: 'cm', name: '厘米' },
  { value: 1, label: 'm', name: '米' }
]
const weightUnits = [
  { value: 0.001, label: 'g', name: '克' },
  { value: 1, label: 'kg', name: '千克' },
  { value: 1000, label: 't', name: '吨' }
]

const radiusUnit = ref(lengthUnits[1]) // 默认厘米
const heightUnit = ref(lengthUnits[1]) // 默认厘米
const thicknessUnit = ref(lengthUnits[0]) // 默认毫米
const priceUnit = ref(weightUnits[2]) // 默认吨

// 材料参数 - 修改为用户输入
const materialName = ref('钢材') // 默认材料名称
const materialDensity = ref('7.85') // 默认密度，g/cm³
const materialPrice = ref('') // 默认价格

// 计算结果
const result = ref(null)
const calculationSteps = ref([])

// 单位转换函数
const convertToMeters = (value, unit) => {
  return parseFloat(value) * unit.value
}

// 计算斜边长度
const calculateSlantHeight = (r1, r2, h) => {
  return Math.sqrt(Math.pow(Math.abs(r1 - r2), 2) + Math.pow(h, 2))
}

// 添加计算步骤
const addStep = (title, formula, value, unit) => {
  calculationSteps.value.push({
    title,
    formula,
    value: value.toFixed(2),
    unit
  })
}

// 计算表面积和重量
const calculate = () => {
  if (!topRadius.value || !bottomRadius.value || !height.value || !thickness.value || !materialPrice.value || !materialDensity.value) {
    return
  }

  calculationSteps.value = [] // 清空之前的计算步骤

  // 转换所有输入为米
  const r1 = convertToMeters(topRadius.value, radiusUnit.value)
  const r2 = convertToMeters(bottomRadius.value, radiusUnit.value)
  const h = convertToMeters(height.value, heightUnit.value)
  const t = convertToMeters(thickness.value, thicknessUnit.value)
  
  let surfaceArea = 0
  if (shape.value === 'frustum') {
    const s = calculateSlantHeight(r1, r2, h)
    addStep('斜边长度', `s = √((r₁ - r₂)² + h²)`, s, 'm')
    
    surfaceArea = Math.PI * (r1 + r2) * s + Math.PI * (r1 * r1 + r2 * r2)
    addStep('表面积计算', `A = π(r₁ + r₂)s + π(r₁² + r₂²)`, surfaceArea, 'm²')
  } else {
    const s = calculateSlantHeight(0, r2, h)
    addStep('斜边长度', `s = √(r² + h²)`, s, 'm')
    
    surfaceArea = Math.PI * r2 * r2 + Math.PI * r2 * s
    addStep('表面积计算', `A = πr² + πrs`, surfaceArea, 'm²')
  }

  // 计算体积(m³)
  const volume = surfaceArea * t
  addStep('体积计算', `V = 表面积 × 厚度 = ${surfaceArea.toFixed(4)} × ${t.toFixed(4)}`, volume, 'm³')
  
  // 计算重量(kg) - 使用用户输入的密度
  const density = parseFloat(materialDensity.value) * 1000 // 转换为 kg/m³ (从 g/cm³)
  const weight = volume * density
  addStep('重量计算', `m = 体积 × 密度 = ${volume.toFixed(4)} × ${density}`, weight, 'kg')
  
  // 计算成本(元)
  const pricePerKg = (parseFloat(materialPrice.value) / priceUnit.value.value) // 转换为元/kg
  const cost = weight * pricePerKg
  addStep('成本计算', `成本 = 重量 × 单价 = ${weight.toFixed(2)} × ${pricePerKg.toFixed(2)}`, cost, '元')

  result.value = {
    surfaceArea: (surfaceArea * 10000).toFixed(2), // 转换为 cm²
    volume: (volume * 1000000).toFixed(2), // 转换为 cm³
    weight: weight.toFixed(2), // kg
    cost: cost.toFixed(2) // 元
  }
}
</script>

<template>
  <div class="absolute-center">
    <div class="calculator-container">
      <!-- 标题区域 -->
      <div class="text-center mb-8">
        <h1 class="text-3xl font-medium text-gray-800 mb-2">几何计算器</h1>
        <p class="text-gray-600">计算圆锥和圆台的表面积、重量和成本</p>
      </div>

      <!-- 主要内容区域 -->
      <div class="bg-white rounded-lg shadow-md p-6">
        <div class="space-y-6">
          <!-- 形状选择 -->
          <div class="tree-item">
            <label class="block text-gray-800 font-medium mb-2">⊞ 形状选择</label>
            <div class="ml-6">
              <select 
                v-model="shape"
                class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
              >
                <option value="frustum">圆锥圆台</option>
                <option value="cone">圆锥</option>
              </select>
            </div>
          </div>

          <!-- 尺寸参数 -->
          <div class="tree-item">
            <label class="block text-gray-800 font-medium mb-2">⊞ 尺寸参数</label>
            <div class="ml-6 space-y-4">
              <div v-if="shape === 'frustum'" class="flex gap-3">
                <div class="flex-1">
                  <label class="block text-gray-700 mb-1">└─ 上底半径</label>
                  <input
                    v-model="topRadius"
                    type="number"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                </div>
                <div class="w-32">
                  <label class="block text-gray-700 mb-1">单位</label>
                  <select
                    v-model="radiusUnit"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                    <option v-for="unit in lengthUnits" :key="unit.label" :value="unit">
                      {{ unit.name }}
                    </option>
                  </select>
                </div>
              </div>

              <div class="flex gap-3">
                <div class="flex-1">
                  <label class="block text-gray-700 mb-1">
                    └─ {{ shape === 'frustum' ? '下底半径' : '底部半径' }}
                  </label>
                  <input
                    v-model="bottomRadius"
                    type="number"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                </div>
                <div class="w-32">
                  <label class="block text-gray-700 mb-1">单位</label>
                  <select
                    v-model="radiusUnit"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                    <option v-for="unit in lengthUnits" :key="unit.label" :value="unit">
                      {{ unit.name }}
                    </option>
                  </select>
                </div>
              </div>

              <div class="flex gap-3">
                <div class="flex-1">
                  <label class="block text-gray-700 mb-1">└─ 高度</label>
                  <input
                    v-model="height"
                    type="number"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                </div>
                <div class="w-32">
                  <label class="block text-gray-700 mb-1">单位</label>
                  <select
                    v-model="heightUnit"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                    <option v-for="unit in lengthUnits" :key="unit.label" :value="unit">
                      {{ unit.name }}
                    </option>
                  </select>
                </div>
              </div>

              <div class="flex gap-3">
                <div class="flex-1">
                  <label class="block text-gray-700 mb-1">└─ 板材厚度</label>
                  <input
                    v-model="thickness"
                    type="number"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                </div>
                <div class="w-32">
                  <label class="block text-gray-700 mb-1">单位</label>
                  <select
                    v-model="thicknessUnit"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                    <option v-for="unit in lengthUnits" :key="unit.label" :value="unit">
                      {{ unit.name }}
                    </option>
                  </select>
                </div>
              </div>
            </div>
          </div>

          <!-- 材料参数 -->
          <div class="tree-item">
            <label class="block text-gray-800 font-medium mb-2">⊞ 材料参数</label>
            <div class="ml-6 space-y-4">
              <div>
                <label class="block text-gray-700 mb-1">└─ 材料名称</label>
                <input
                  v-model="materialName"
                  type="text"
                  class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  placeholder="请输入材料名称"
                >
              </div>
              
              <div>
                <label class="block text-gray-700 mb-1">└─ 材料密度 (g/cm³)</label>
                <input
                  v-model="materialDensity"
                  type="number"
                  step="0.01"
                  class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  placeholder="请输入材料密度"
                >
              </div>

              <div class="flex gap-3">
                <div class="flex-1">
                  <label class="block text-gray-700 mb-1">└─ 材料价格</label>
                  <input
                    v-model="materialPrice"
                    type="number"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                    placeholder="请输入价格"
                  >
                </div>
                <div class="w-32">
                  <label class="block text-gray-700 mb-1">单位</label>
                  <select
                    v-model="priceUnit"
                    class="w-full p-2.5 bg-white border-2 border-gray-300 rounded-md focus:border-blue-500 focus:ring-0 text-gray-700"
                  >
                    <option v-for="unit in weightUnits" :key="unit.label" :value="unit">
                      元/{{ unit.label }}
                    </option>
                  </select>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 计算按钮 -->
        <button
          @click="calculate"
          class="w-full mt-8 py-3 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors font-medium"
        >
          计算结果
        </button>

        <!-- 结果显示 -->
        <div v-if="result" class="mt-8 space-y-6">
          <!-- 最终结果 -->
          <div class="tree-item">
            <label class="block text-gray-800 font-medium mb-2">⊞ 计算结果</label>
            <div class="ml-6 space-y-2 text-gray-700">
              <p>└─ 表面积: {{ result.surfaceArea }} cm²</p>
              <p>└─ 体积: {{ result.volume }} cm³</p>
              <p>└─ 重量: {{ result.weight }} kg</p>
              <p>└─ 成本: {{ result.cost }} 元</p>
            </div>
          </div>

          <!-- 计算步骤 -->
          <div class="tree-item">
            <label class="block text-gray-800 font-medium mb-2">⊞ 计算过程</label>
            <div class="ml-6 space-y-4">
              <div v-for="(step, index) in calculationSteps" :key="index" class="text-gray-700">
                <p class="font-medium">└─ {{ step.title }}:</p>
                <p class="ml-6">{{ step.formula }}</p>
                <p class="ml-6">= {{ step.value }} {{ step.unit }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* 重置基础样式 */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html,
body,
#app {
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
}

/* 绝对居中布局 - 关键样式 */
.absolute-center {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f3f4f6;
}

.calculator-container {
  width: 100%;
  max-width: 768px;
  padding: 1.5rem;
  overflow-y: auto;
  max-height: 100vh;
}

/* 输入框样式 */
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}

/* 树形结构样式 */
.tree-item {
  position: relative;
}

.tree-item::before {
  content: '';
  position: absolute;
  left: 0;
  top: 2rem;
  bottom: 0.5rem;
  width: 1px;
  background-color: #e5e7eb;
  display: none;
}

.tree-item:hover::before {
  display: block;
}
</style>
