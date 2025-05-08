<template>
  <el-row :gutter="20">
    <el-col :span="8">
      <pre>{{ form.skuList }}</pre>
    </el-col>
    <el-col :span="16">
      <div>

        <div v-if="!disabled" :class="`sku-form-container-${theme}`" class="sku-form-container">
          <el-card style="margin-bottom: 20px">
            <div v-for="(pkgItem, pkgIndex) in myPackage" :key="pkgIndex" class="" style="display: flex;flex-flow: column;margin-bottom: 30px">
              <div style="display: flex;align-items: center;">
                <el-checkbox v-model="pkgItem.checked"/>

                <div class="sku-form-title">
                  <el-input
                      v-model="pkgItem.name"
                      placeholder="请输入套餐名称"
                  />
                </div>
              </div>
              <template v-if="pkgItem.checked">
                <view style="display: flex;justify-content: space-between">
                  <div>
                    买断模式:
                    <el-radio-group v-model="pkgItem.buyout_mode">
                      <el-radio :value="0">不可买断</el-radio>
                      <el-radio :value="1">提前买断</el-radio>
                      <el-radio :value="2">到期买断</el-radio>
                    </el-radio-group>
                  </div>
                  <div v-if="pkgItem.buyout_mode != 0">
                    买断折扣：
                    <el-input-number v-model="pkgItem.buyout_discount" :max="100" :min="1"/>
                  </div>
                  <div>
                    是否可续租：
                    <el-switch v-model="pkgItem.is_relet" :active-value="1" :inactive-value="0"/>
                  </div>
                  <div v-if="pkgItem.is_relet">
                    续租系数：
                    <el-input-number v-model="pkgItem.relet_coefficient" :max="2" :min="1"/>
                  </div>
                  <div style="width: 200px">
                    <el-select
                        v-model="pkgItem.selected_rent_duration"
                        multiple
                        placeholder="请选择租期"
                        @change="onSelectedChange(pkgItem)"
                    >
                      <el-option
                          v-for="duration in pkgItem.rent_duration"
                          :key="duration"
                          :label="`${duration}${pkgItem.unit}`"
                          :value="duration"
                      />
                    </el-select>
                  </div>
                </view>
              </template>
            </div>
          </el-card>
          <el-card>
            <div style="text-align: right;margin-bottom: 10px">
              <el-button @click="addSpec">添加规格</el-button>
            </div>
            <div v-for="(attrItem, attrIndex) in myAttribute" :key="attrIndex" class="sku-form-section">
              <div class="sku-form-title" style="margin-right: 50px">
                <el-input
                    v-model="attrItem.name"
                    placeholder="请输入规格名称"
                    size="small"
                >
                  <template #prepend>
                    <el-checkbox v-model="attrItem.has_image" label="是否有图"/>
                  </template>
                </el-input>
              </div>
              <div class="sku-form-tags-box">

                <div v-for="(item, index) in attrItem.items" :key="index"
                     class="sku-checkbox-content">
                  <img
                      v-if="attrItem.has_image"
                      :alt="item.name"
                      :src="item.image"
                      class="sku-option-image"
                  />
                  <el-input
                      v-model="item.value"
                      placeholder="请输入规格值"
                      size="small"
                  />
                  <el-icon class="sku-option-delete" @click="() => onDeleteAttributeItem(attrIndex, index)">
                    X
                  </el-icon>

                </div>
              </div>
              <div class="sku-form-add-tags">
                <el-input
                    v-model="inputValues[attrIndex]"
                    placeholder="请输入规格名称"
                    size="small"
                    @keyup.enter="onAddAttribute(attrIndex)"
                >
                  <template #append>
                    <el-button :icon="Plus" @click="onAddAttribute(attrIndex)">添加</el-button>
                  </template>
                </el-input>
              </div>
            </div>
          </el-card>
        </div>
        <!-- @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ -->
        <el-form
            ref="formRef"
            :class="disabled ? 'sku-form-table-disabled' : ''"
            :model="form"
            :rules="rules"
            class="sku-form-table"
        >
          <div v-for="(pkgItem, pkgIndex) in form.skuList" :key="pkgIndex" class="sku-form-section">
            <template v-if="pkgItem.pkg.checked">
              <h4>{{ pkgItem.pkg.name }}</h4>
              <el-table
                  :key="pkgItem.pkg.skus.length"
                  :data="pkgItem.pkg.skus"
                  border
                  style="width: 100%"
              >
                <el-table-column label="SKU" prop="sku"></el-table-column>

                <el-table-column
                    v-for="(col, colIndex) in structure"
                    :key="colIndex"
                    :label="col.label"
                    align="center"
                >
                  <template v-if="col.batch !== false && col.type === 'input' && isBatch" #header>
                    <div class="sku-form-batch">
                      <el-input v-model="batch[col.name]" :placeholder="`统一设置${col.label}`" size="small">
                        <template #append>
                          <el-button @click="onBatchSet(col.name)">批量设置</el-button>
                        </template>
                      </el-input>
                    </div>
                  </template>

                  <template #default="{ row, $index }">
                    <div v-if="col.type === 'slot'">
                      <slot :index="$index" :name="col.name" :row="row"/>
                    </div>
                    <div v-if="col.type === 'switch'">
                      <el-switch v-model="row[col.name]" :active-value="1" :inactive-value="0"/>
                    </div>
                    <el-form-item
                        v-else
                        :class="`sku-form-${$index}-${col.name}`"
                        :prop="`skuData.${$index}.${col.name}`"
                    >
                      <el-tooltip
                          v-if="col.tips"
                          :content="col.tips"
                          :hide-after="0"
                          placement="top"
                      >
                        <el-icon class="sku-form-tips">
                          <InfoFilled/>
                        </el-icon>
                      </el-tooltip>
                      <el-input
                          v-model="row[col.name]"
                          :disabled="col.disabled"
                          :placeholder="col.placeholder"
                          size="small"
                      />
                    </el-form-item>
                  </template>
                </el-table-column>


                <!-- pkg -->
                <el-table-column
                    v-for="(aa, dIndex) in pkgItem.selected_rent_duration"
                    :key="dIndex"
                    align="center"
                >
                  <template #header>{{ aa }}{{ pkgItem.unit }}</template>
                  <template #default="{ row, $index }">
                    <el-input

                        size="small"
                    />
                  </template>
                </el-table-column>


                <el-table-column>
                  <template #header>debug</template>
                  <template #default="{ row, $index }">
                    <pre>row</pre>
                  </template>
                </el-table-column>


              </el-table>
            </template>
          </div>
        </el-form>
      </div>
    </el-col>

  </el-row>

</template>

<script setup>
import {computed, nextTick, onMounted, reactive, ref, toRefs, unref, watch} from 'vue'
import {InfoFilled, Plus} from '@element-plus/icons-vue'
import {ElMessage, ElMessageBox} from 'element-plus'

const props = defineProps({
  /**
   * 原始规格数据
   * sourceAttribute: [
   *   { name: '颜色', item: ['黑', '金', '白'] },
   *   { name: '内存', item: ['16G', '32G'] },
   *   { name: '运营商', item: ['电信', '移动', '联通'] }
   * ]
   */
  sourceAttribute: {
    type: Array,
    default: () => []
  },
  /**
   * 已使用的规格数据，用于复原数据，支持v-model:attribute修饰符
   * attribute: [
   *   { name: '颜色', item: ['黑'] },
   *   { name: '运营商', item: ['电信', '移动', '联通'] }
   * ]
   */
  attribute: {
    type: Array,
    default: () => []
  },
  //
  sourcePackage: {
    type: Array,
    default: () => [
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
    ]
  },
  pkg: {
    type: Array,
    default: () => []
  },
  /**
   * 用于复原sku数据，支持v-model:sku修饰符
   * sku: [
   *   { sku: '黑;电信', price: 1, stock: 1 },
   *   { sku: '黑;移动', price: 2, stock: 2 },
   *   { sku: '黑;联通', price: 3, stock: 3 }
   * ]
   */
  sku: {
    type: Array,
    default: () => []
  },
  /**
   * 表格结构，注意name字段，用于输出sku数据
   */
  structure: {
    type: Array,
    default: () => [
      {name: 'status', type: 'switch', label: '状态'},
      {name: 'price', type: 'input', label: '价格'},
      {name: 'stock', type: 'input', label: '库存'},
    ]
  },
  // sku 字段分隔符
  separator: {
    type: String,
    default: ';'
  },
  // 无规格的 sku
  emptySku: {
    type: String,
    default: ''
  },
  // 是否显示 sku 选择栏
  disabled: {
    type: Boolean,
    default: false
  },
  // 主题风格
  theme: {
    type: Number,
    default: 2
  },
  // 是否开启异步加载
  async: {
    type: Boolean,
    default: false
  },
  // 是否可添加属性值
  canAddAttribute: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['update:pkg', 'update:attribute', 'update:sku', 'validate'])

// 使用toRefs优化props解构，保持响应性
const {sourceAttribute, attribute, sourcePackage, pkg, sku, structure, separator, emptySku, async: isAsync, canAddAttribute} = toRefs(props)

// 表单引用
const formRef = ref(null)
const isInit = ref(false)

// 输入框的值独立管理，避免深层次响应式问题
const inputValues = ref([])

// 数据
const form = reactive({
  skuData: [],
  skuList: [],
})

// 批量设置暂存数据
const batch = reactive({})

// 属性数据(包含选中状态)
const myAttribute = ref([])

// 套餐数据(包含选中状态)
const myPackage = ref([])


// 计算规则
const rules = computed(() => {
  const result = {}
  structure.value.forEach(item => {
    if (item.required) {
      const rule = {required: true, message: `请输入${item.label}`, trigger: 'blur'}
      if (item.validator) {
        rule.validator = item.validator
      }
      result[item.name] = [rule]
    } else if (item.validator) {
      result[item.name] = [{validator: item.validator, trigger: 'blur'}]
    }
  })
  return result
})

// 是否显示批量设置
const isBatch = computed(() => {
  return structure.value.some(item => item.type === 'input' && item.batch !== false)
})

// 仅输出勾选的属性
const emitAttribute = computed(() => {
  return myAttribute.value.map(attr => ({
    ...attr,
    items: attr.items.filter(item => item.value.trim() !== '')
  }))
})

const emitPackage = computed(() => {
  return myPackage.value.filter(f => f.checked).map(item => {
    const _pkg = {
      ...item,
      rent_duration: item.selected_rent_duration
    }
    delete _pkg.selected_rent_duration
    delete _pkg.checked
    return _pkg;
  });
})

// 初始化方法
const init = () => {
  nextTick(() => {
    isInit.value = true

    // 初始化 myAttribute
    myAttribute.value = JSON.parse(JSON.stringify(attribute.value))

    // ----------------------------------------------------------------------------------------------------
    // 初始化 myPackage
    const newMyPackage = []
    // 根据 sourcePackage 复原 myPackage 的结构
    sourcePackage.value.forEach(v => {
      const temp = JSON.parse(JSON.stringify(v))
      temp.selected_rent_duration = []
      temp.checked = false

      pkg.value.forEach(attrVal => {
        if (attrVal.rent_mode === v.rent_mode) {
          Object.keys(attrVal).forEach(key => {
            if (key !== 'skus') {
              temp[key] = JSON.parse(JSON.stringify(attrVal[key]))
            }
          })

          let newRentDuration = [...new Set([...v.rent_duration, ...attrVal.rent_duration])].sort((a, b) => a - b)
          temp.rent_duration = newRentDuration
          temp.checked = true
          temp.selected_rent_duration = JSON.parse(JSON.stringify(attrVal.rent_duration))
          temp.skus = [];
        }
      });
      newMyPackage.push(temp)
      form.skuList.push({pkg: JSON.parse(JSON.stringify(temp))})
    })
    myPackage.value = newMyPackage
    console.log('sdfasfd', newMyPackage)

    // ----------------------------------------------------------------------------------------------------

    // 因为 skuData 是实时监听 myAttribute 变化并自动生成，使用微任务确保已生成
    setTimeout(() => {
      console.log('开始复原SKU数据.....', sku.value)
      form.skuData = JSON.parse(JSON.stringify(sku.value))

      form.skuList.forEach(v => {
        sku.value.forEach(skuItem => {
          v.pkg.spec = myAttribute.value;
          if (skuItem.pkg?.rent_mode === v.pkg.rent_mode) {
            v.pkg.skus.push(JSON.parse(JSON.stringify(skuItem)))
          }
        })
      })

      isInit.value = false
    }, 0)
  })
}

onMounted(() => {
  console.log('初始化', isAsync.value)
  if (!isAsync.value) {
    init()
  }
})


// 监听选中属性的变化
watch(myAttribute, () => {
  console.log('watch.myAttribute 属性值变化', myAttribute.value)
  if (!isInit.value) {
    // 更新父组件
    emit('update:attribute', emitAttribute.value)
  }
  // 解决通过 $emit 更新后无法拿到 attribute 最新数据的问题
  nextTick(() => {
    if (emitAttribute.value.length !== 0) {
      combinationAttribute()
    } else {
      form.skuData = []
      const obj = {
        sku: emptySku.value,
      }
      structure.value.forEach(v => {
        if (!(v.type === 'slot' && v.skuProperty === false)) {
          obj[v.name] = typeof v.defaultValue !== 'undefined' ? v.defaultValue : ''
        }
      })
      form.skuData.push(obj)
    }
    clearValidate()
  })
}, {deep: true})

watch(myPackage, () => {
  if (!isInit.value) {
    // 更新父组件
    emit('update:pkg', emitPackage.value)
  }
  nextTick(() => {
    combinationPackage()
    clearValidate()
  })
}, {deep: true})


// 监听skuData变化
watch(() => form.skuData, (newValue, oldValue) => {
  if (!isInit.value || (newValue.length === 1 && newValue[0].sku === emptySku.value)) {
    // 如果有老数据，或者 sku 数据为空，则更新父级 sku 数据
    if (oldValue.length || !sku.value.length) {
      // 更新父组件
      const arr = [];
      newValue.forEach(v1 => {
        const obj = {
          sku: v1.sku,
          skuData: v1.skuData || {} // 完整的SKU数据
        };

        structure.value.forEach(v2 => {
          if (!(v2.type === 'slot' && v2.skuProperty === false)) {
            obj[v2.name] = v1[v2.name] || (typeof v2.defaultValue !== 'undefined' ? v2.defaultValue : '');
          }
        });

        arr.push(obj);
      });

      emit('update:sku', arr);
    }
  }
}, {deep: true})

// 组合属性，生成SKU表格数据
const combinationAttribute = () => {
  const attrs = emitAttribute.value;
  let result = [];

  if (attrs.length === 0) {
    // 处理无属性的情况
    form.skuData = [];
    return;
  }

  // 利用 reduce 实现笛卡尔积
  result = attrs.reduce((acc, attr) => {
    const items = attr.items.map(item => item.value);
    if (acc.length === 0) {
      return items.map(val => ({
        sku: val,
        spec: [{name: attr.name, value: val}],
      }));
    }
    return acc.flatMap(prev =>
        items.map(val => ({
          sku: `${prev.sku}${separator.value}${val}`,
          spec: [
            ...prev.spec,
            {name: attr.name, value: val},
          ],
        }))
    );
  }, []);

  // --
  result.forEach(item => {
    structure.value.forEach(v => {
      if (!(v.type === 'slot' && v.skuProperty === false)) {
        item[v.name] = typeof v.defaultValue !== 'undefined' ? v.defaultValue : ''
      }
    })
  })

  console.log('reduce.result', result)

  form.skuData = result;
};

const rent_duration = ref({})
const combinationPackage = () => {
  console.log('开始combinationPackage数据.....', emitPackage.value)

  let result = []
  emitPackage.value.forEach(v => {
    rent_duration.value['rent_mode_' + v.rent_mode] = v.rent_duration
  });
  // rent_duration.value = result
  console.log('rent_duration', rent_duration.value)
}

// 查找属性的详细信息
const findAttributeDetail = (attrName, itemName) => {
  const attrGroup = myAttribute.value.find(attr => attr.name === attrName);
  if (!attrGroup) return {name: itemName};

  const item = attrGroup.item.find(item => item.name === itemName);
  if (!item) return {name: itemName};

  // 返回完整的属性对象，仅过滤掉checked状态
  const {checked, ...itemDetail} = item;
  return itemDetail;
}

// 添加新属性值
const onAddAttribute = (index) => {
  const newValue = inputValues.value[index]?.trim();

  if (!newValue) {
    ElMessage.warning('请输入规格名称');
    return;
  }

  // 检查分隔符
  if (newValue.includes(separator.value)) {
    ElMessageBox.alert(`规格里不允许出现「 ${separator.value} 」字符，请检查后重新添加`, '警告', {
      confirmButtonText: '确定',
      type: 'warning'
    });
    return;
  }

  // 检查最大长度
  if (newValue.length > 50) {
    ElMessage.warning('规格名称长度不能超过50个字符');
    return;
  }

  // 检查重复
  if (myAttribute.value[index].items.some(item => item.name === newValue)) {
    ElMessage.warning('请勿添加相同规格');
    return;
  }

  console.log('myAttribute.value[index]', myAttribute.value[index])
  // 添加新属性，并默认选中


  let _obj = {
    value: newValue,
  };
  let hasImg = myAttribute.value[index].has_image
  if (hasImg) {
    _obj.image = ''
  }
  myAttribute.value[index].items.push(_obj);

  // 清空输入框
  inputValues.value[index] = '';
}

// 批量设置
const onBatchSet = (field) => {
  if (batch[field] !== '') {
    form.skuData.forEach(row => {
      row[field] = batch[field]
    })

    batch[field] = ''
  }
}

// 表单验证
const validate = (callback) => {
  if (formRef.value) {
    formRef.value.validate(valid => {
      callback && callback(valid)
    })
  } else {
    callback && callback(false)
  }
}

// 自定义验证
const validateFieldByColumns = (columns, callback) => {
  if (!formRef.value) {
    callback && callback(false)
    return
  }

  const propPaths = []
  form.skuData.forEach((_, i) => {
    columns.forEach(col => {
      propPaths.push(`skuData.${i}.${col}`)
    })
  })

  formRef.value.validateField(propPaths, valid => {
    callback && callback(valid)
  })
}

// 按行验证
const validateFieldByRows = (index, prop, callback) => {
  if (formRef.value) {
    formRef.value.validateField([`skuData.${index}.${prop}`], valid => {
      callback && callback(valid)
    })
  } else {
    callback && callback(false)
  }
}

// 清除验证
const clearValidate = () => {
  if (formRef.value) {
    formRef.value.clearValidate()
  }
}

// 添加新属性（带图片）
const onAddAttributeWithImage = (index, name, imagePath) => {
  if (!name || typeof name !== 'string') {
    ElMessage.warning('请提供有效的规格名称');
    return;
  }

  const newValue = name.trim();

  if (!newValue) {
    ElMessage.warning('规格名称不能为空');
    return;
  }

  // 检查最大长度
  if (newValue.length > 50) {
    ElMessage.warning('规格名称长度不能超过50个字符');
    return;
  }

  // 检查分隔符
  if (newValue.includes(separator.value)) {
    ElMessageBox.alert(`规格里不允许出现「 ${separator.value} 」字符，请检查后重新添加`, '警告', {
      confirmButtonText: '确定',
      type: 'warning'
    });
    return;
  }

  // 检查重复
  if (myAttribute.value[index].item.some(item => item.name === newValue)) {
    ElMessage.warning('请勿添加相同规格');
    return;
  }

  // 添加新属性（带图片），并默认选中
  myAttribute.value[index].item.push({
    name: newValue,
    image: imagePath || '',
  });

  return true;
}


// 处理checkbox变化
const onCheckedChange = (attrIndex, itemIndex, isChecked) => {
  // 更新原始item的checked状态
  myAttribute.value[attrIndex].item[itemIndex].checked = isChecked
}

const onSelectedChange = (pkgItem) => {
  console.log(pkgItem)
}

const addSpec = () => {
  console.log('addSpec')
  myAttribute.value.push({
    name: '规格' + (myAttribute.value.length + 1),
    canAddAttribute: true,
    item: [
      {
        name: '规格1',
        checked: false
      }
    ]
  });
}

const onDeleteAttributeItem = (attrIndex, index) => {
  myAttribute.value[attrIndex].item.splice(index, 1);
}

// 获取checked状态
const getCheckedStatus = (pkgItem) => {
  return !!pkgItem.checked;
}

// 暴露方法
defineExpose({
  init,
  validate,
  validateFieldByColumns,
  validateFieldByRows,
  clearValidate,
  onAddAttributeWithImage
})
</script>

<style lang="scss" scoped>
.sku-form {
  &-container {
    margin-bottom: 10px;

    &-1 {
      .sku-form-section {
        margin-bottom: 10px;
      }

      .sku-form-title {
        margin-bottom: 10px;
        font-weight: bold;
      }

      .sku-form-tags-box {
        margin-bottom: 10px;

        .checkbox-group {
          display: flex;
          flex-wrap: wrap;
          gap: 8px;
        }
      }
    }

    &-2 {
      padding: 10px;
      border: 1px solid #ebeef5;
      border-radius: 4px;

      .sku-form-section {
        display: flex;
        flex-wrap: wrap;
        align-items: center;
        margin-bottom: 10px;

        &:last-child {
          margin-bottom: 0;
        }
      }

      .sku-form-title {
        min-width: 70px;
        margin-right: 10px;
        font-weight: bold;
      }

      .sku-form-tags-box {
        flex: 1;
        margin-right: 10px;

        .checkbox-group {
          display: flex;
          flex-wrap: wrap;
          gap: 8px;
        }
      }

      .sku-form-add-tags {
        width: 230px;
      }
    }
  }

  &-table {
    :deep(.el-table .cell) {
      padding: 0 5px;
    }

    :deep(.el-form-item) {
      margin-bottom: 0;
    }

    :deep(.el-form-item__content) {
      min-height: 32px;
      display: flex;
      align-items: center;
    }

    &-disabled {
      .el-table {
        pointer-events: none;
      }
    }
  }

  &-batch {
    margin-bottom: 5px;
  }

  &-tips {
    margin-right: 5px;
    color: #409eff;
    cursor: pointer;
  }
}

// 新增样式 - 规格选项相关
.sku-checkbox-content {
  display: flex;
  align-items: center;
  gap: 4px;
}

.sku-option-image {
  width: 24px;
  height: 24px;
  object-fit: cover;
  border-radius: 2px;
}

.sku-table-cell {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
}

.sku-table-image {
  width: 20px;
  height: 20px;
  object-fit: cover;
  border-radius: 2px;
}
</style>
