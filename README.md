# vue-sku-form

基于 Vue3 & Element Plus 的电商 SKU 表单配置组件

这是[vue-sku-form](https://github.com/hooray/vue-sku-form)的Vue3版本，完全兼容原有的功能和API。

## ✨ 特性

- 基于 Vue3 Composition API 和 Element Plus
- 完全兼容原 vue-sku-form 的 API
- 支持使用 v-model:attribute 和 v-model:sku 双向绑定
- 🚀 支持多属性 SKU 组合，自动进行笛卡尔积计算
- ✅ 支持属性选择和添加新属性
- 📊 支持批量设置 SKU 数据
- 🔍 内置表单验证功能
- 🎨 支持多种主题风格
- 🧩 丰富的插槽支持，自定义任意表单项

## 安装

```bash
npm install vue-sku-form
```

## 使用

### 全局注册

```js
import { createApp } from 'vue'
import App from './App.vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import SkuForm from 'vue-sku-form'

const app = createApp(App)
app.use(ElementPlus)
app.use(SkuForm)
app.mount('#app')
```

### 局部注册

```vue
<template>
  <SkuForm
    ref="skuFormRef"
    :source-attribute="sourceAttribute"
    v-model:attribute="attribute"
    v-model:sku="sku"
    :structure="structure"
  >
    <template #custom-field="{ row, index }">
      <!-- 自定义内容 -->
      <el-upload>
        <el-button>上传图片</el-button>
      </el-upload>
    </template>
  </SkuForm>
  <el-button @click="validate">验证</el-button>
</template>

<script setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import SkuForm from 'vue-sku-form'

const skuFormRef = ref(null)
const sourceAttribute = ref([
  { name: '颜色', item: ['红色', '蓝色', '黑色'] },
  { name: '尺寸', item: ['S', 'M', 'L', 'XL'] }
])
const attribute = ref([])
const sku = ref([])
const structure = ref([
  { 
    name: 'price', 
    type: 'input', 
    label: '价格', 
    required: true,
    validator: (rule, value, callback) => {
      if (value === '') {
        callback(new Error('价格不能为空'))
      } else if (isNaN(Number(value))) {
        callback(new Error('价格必须为数字'))
      } else if (Number(value) <= 0) {
        callback(new Error('价格必须大于0'))
      } else {
        callback()
      }
    }
  },
  { name: 'stock', type: 'input', label: '库存(件)', required: true },
  { name: 'weight', type: 'input', label: '重量(KG)' },
  { name: 'image', type: 'slot', label: '图片' },
  { name: 'status', type: 'slot', label: '状态' }
])

const validate = () => {
  skuFormRef.value.validate((valid) => {
    if (valid) {
      ElMessage.success('验证通过')
    } else {
      ElMessage.error('验证失败')
    }
  })
}
</script>
```

## 参数说明

| 参数 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| sourceAttribute | 原始规格数据源 | Array | [] |
| attribute | 已选中的规格数据，支持 v-model | Array | [] |
| sku | 生成的SKU表格数据，支持 v-model | Array | [] |
| structure | 表格结构配置 | Array | [{ name: 'price', type: 'input', label: '价格' }, { name: 'stock', type: 'input', label: '库存' }] |
| separator | SKU代码分隔符 | String | ';' |
| emptySku | 无规格时的SKU代码 | String | '' |
| disabled | 是否禁用属性选择 | Boolean | false |
| theme | 主题风格，支持 1 和 2 两种风格 | Number | 1 |
| async | 是否开启异步加载 | Boolean | false |
| canAddAttribute | 是否可添加新属性值 | Boolean | true |

## 方法

| 方法名 | 说明 | 参数 |
| --- | --- | --- |
| init | 手动初始化组件 | - |
| validate | 表单验证 | (callback: Function) |
| validateFieldByColumns | 按列验证表单 | (columns: Array, callback: Function) |
| validateFieldByRows | 按行验证表单 | (index: Number, prop: String, callback: Function) |
| clearValidate | 清除验证结果 | - |

## 事件

| 事件名 | 说明 | 参数 |
| --- | --- | --- |
| update:attribute | 当属性选择变化时触发 | 已选属性数组 |
| update:sku | 当生成的SKU数据变化时触发 | SKU数据数组 |
| validate | 表单验证结果事件 | 验证是否通过布尔值 |

## 插槽

组件支持自定义插槽，用于自定义表单项的渲染方式。在 `structure` 配置中设置 `type: 'slot'` 并提供对应的插槽名称，即可使用自定义插槽。

```vue
<SkuForm>
  <template #image="{ row, index }">
    <el-upload>
      <el-button>上传图片</el-button>
    </el-upload>
  </template>
  <template #status="{ row, index }">
    <el-switch v-model="row.status" />
  </template>
</SkuForm>
```

## 样式主题

组件支持两种主题风格，通过 `theme` 属性进行切换：

```vue
<template>
  <el-radio-group v-model="theme">
    <el-radio :label="1">主题1</el-radio>
    <el-radio :label="2">主题2</el-radio>
  </el-radio-group>
  <SkuForm
    :source-attribute="sourceAttribute"
    v-model:attribute="attribute"
    v-model:sku="sku"
    :theme="theme"
  />
</template>

<script setup>
import { ref } from 'vue'
import SkuForm from 'vue-sku-form'

const theme = ref(1)
const sourceAttribute = ref([
  // ...
])
const attribute = ref([])
const sku = ref([])
</script>
```

## 开发

```bash
npm run dev
```

## 构建

```bash
npm run build
```

## 许可证

MIT

---

本组件基于 Vue 3 和 Element Plus 开发，适用于电商、企业管理系统等需要进行SKU规格设置的场景。如有问题或建议，欢迎提交 issue 或 PR。


