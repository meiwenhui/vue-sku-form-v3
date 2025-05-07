<template>
  <div>
    <SkuForm
        v-model:pkg="pkg"
        v-model:attribute="attribute"
        v-model:sku.sync="sku"/>
    <el-row :gutter="20">
      <el-col :span="8">
        <pre><code>{{ pkg }}</code></pre>
      </el-col>
      <el-col :span="8">
        <pre><code>{{ attribute }}</code></pre>
      </el-col>
      <el-col :span="8">
        <pre><code>{{ sku }}</code></pre>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import {ref} from 'vue'
import SkuForm from '../components/SkuForm.vue'
import data from './data.json'

const attribute = ref(data.spec)
const pkg = ref(data.pkg)
const sku = ref(data.sku)
</script>
