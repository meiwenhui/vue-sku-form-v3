<template>
  <div>
    <SkuForm
        v-model:attribute="attribute"
        :source-attribute="sourceAttribute"
        :structure="structure"
        v-model:sku.sync="sku"
        :source-package="sourcePackage"
        :pkg="pkg"
    >
      <template #status="slotProps">
        <div>
          <el-switch v-model="slotProps.row.status" :active-value="1" :inactive-value="0"/>
        </div>
      </template>
    </SkuForm>
    <el-row :gutter="20">
      <el-col :span="8">
        <el-divider content-position="left">attribute 数据</el-divider>
        <pre><code>{{ attribute }}</code></pre>
      </el-col>
      <el-col :span="8">
        <el-divider content-position="left">sku 数据</el-divider>
        <pre><code>{{ sku }}</code></pre>
      </el-col>
      <el-col :span="8">
        <el-divider content-position="left">package 数据</el-divider>
        <pre><code>{{ pkg }}</code></pre>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import {ref} from 'vue'
import SkuForm from '../components/SkuForm.vue'

const sourcePackage = ref([
  {
    "unit": "天", // 租期单位
    "checked": false, // 租期单位
    "name": "租完归还",  // 租期名称
    "buyout_mode": 1, // 买断模式； 可选：0：不可买断；1：提前买断；2：到期买断
    "is_relet": 0,// 是否可续租； 可选：0：不可续租；1：可续租
    "rent_mode": 2, // 租赁模式； 可选：1：租完即送；2：灵活租
    "buyout_discount": 100,  // 买断折扣； 当  buyout_mode != 1时显示，取值范围 0-100
    "relet_coefficient": 1, // 续租系数； 当  is_relet = 1时显示，取值范围 1-2
    "rent_duration": [ // 可租赁时间列表
      180, 365
    ]
  },
  {
    "checked": false,
    "unit": "天", // 租期单位
    "name": "灵活租",  // 租期名称
    "buyout_mode": 1, // 买断模式； 可选：0：不可买断；1：提前买断；2：到期买断
    "is_relet": 1,// 是否可续租； 可选：0：不可续租；1：可续租
    "rent_mode": 3, // 租赁模式； 可选：1：租完即送；2：灵活租
    "buyout_discount": 100,  // 买断折扣； 当  buyout_mode != 1时显示，取值范围 0-100
    "relet_coefficient": 1, // 续租系数； 当  is_relet = 1时显示，取值范围 1-2
    "rent_duration": [ // 可租赁时间列表
      180, 365
    ]
  }
])

const pkg = ref([
  {
    "unit": "天",
    "name": "租完可归还/买断/续租",
    "buyout_mode": 1,
    "is_relet": 1,
    "rent_mode": 2,
    "buyout_discount": 100,
    "relet_coefficient": 1.5,
    "rent_duration": [
      90, 365
    ]
  }
])

const sourceAttribute = ref([
  {
    name: '颜色',
    item: ['黑', '金', '白']
  },
  {
    name: '内存',
    item: ['16G', '32G']
  }
])
const attribute = ref([
  {
    name: '颜色',
    item: ['黑']
  },
  {
    name: '内存',
    item: ['16G']
  }
])
const structure = ref([
  {
    name: 'status',
    type: 'slot',
    defaultValue: 0,
    label: '状态'
  },
  {
    name: 'originalprice',
    type: 'input',
    label: '原价'
  },
  {
    name: 'price',
    type: 'input',
    label: '现价'
  },
  {
    name: 'stock',
    type: 'input',
    label: '库存'
  },
])

const sku = ref([
  {
    "sku": "黑;16G",
    "originalprice": 200,
    "price": 82,
    "stock": 200,
    "status": 0,
    'spec': [
      {
        "attr_key": "颜色",
        "attr_value": "黑"
      },
      {
        "attr_key": "内存",
        "attr_value": "16G"
      },
    ],
    "pkg": {
      "unit": "天",
      "name": "租完归还",
      "buyout_mode": 1,
      "is_relet": 0,
      "rent_mode": 2,
      "buyout_discount": 100,
      "relet_coefficient": 1,
      "duration_price_list": [
        {
          "duration": 365,
          "total_sale_price": 3100,
          "total_rent_price": 2200
        },
        {
          "duration": 90,
          "total_sale_price": 1200,
          "total_rent_price": 2400
        }
      ]
    }
  },
])
</script>
