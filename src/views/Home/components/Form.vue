<template>
  <div class="home-right">
    <el-form
        :label-position="labelPosition"
        label-width="100px"
        ref="ruleForm"
        :rules="rules"
        :model="form"
        style="max-width: 460px"
    >
      <el-row :gutter="20">
        <el-col :span="24">
          <el-form-item label="类型">
            <el-radio v-model="form.choose" :label="1" @change="chooseChange" size="large">小稀有</el-radio>
            <el-radio v-model="form.choose" :label="2" @change="chooseChange" size="large">大稀有</el-radio>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <el-form-item label="总幸运值" prop="lucky">
            <el-input-number :min="0" v-model="form.lucky"/>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <template v-if="form.choose===1">
            <el-form-item label="纳地基础所需数量" prop="landBase">
              <el-input-number :min="0" v-model="form.landBase"/>
            </el-form-item>
          </template>
          <template v-else>
            <el-form-item label="承天基础所需数量" prop="skyBase">
              <el-input-number :min="0" v-model="form.skyBase"/>
            </el-form-item>
          </template>
        </el-col>
        <el-col :span="12">
          <el-form-item label="百炼装备幸运" prop="high">
            <el-input-number :min="0" v-model="form.high"/>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="普紫装备幸运" prop="average">
            <el-input-number :min="0" v-model="form.average"/>
          </el-form-item>
        </el-col>
        <!--        <el-col :span="12">-->
        <!--          <el-form-item label="纳地上限" prop="landMax">-->
        <!--&lt;!&ndash;            <el-input-number :min="0" v-model="form.landMax"/>&ndash;&gt;-->
        <!--            {{form.landMax}}-->
        <!--          </el-form-item>-->
        <!--        </el-col>-->
        <!--        <el-col :span="12">-->
        <!--          <el-form-item label="承天上限" prop="skyMax">-->
        <!--&lt;!&ndash;            <el-input-number :min="0" v-model="form.skyMax"/>&ndash;&gt;-->
        <!--            {{form.skyMax}}-->
        <!--          </el-form-item>-->
        <!--        </el-col>-->
        <el-col :span="12">
          <el-form-item label="纳地单次幸运" prop="landLucky">
            <el-input-number :min="0" v-model="form.landLucky"/>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="承天单次幸运" prop="skyLucky">
            <el-input-number :min="0" v-model="form.skyLucky"/>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <p>排序计算比例：百炼:承天:纳地=1:1:5</p>
          <p>PS：1个百炼等于1个承天等于5个纳地</p>
        </el-col>
        <el-col :span="24">
          <el-form-item label="综合排序" prop="checkedType">
            <el-checkbox-group v-model="form.checkedType">
              <el-checkbox v-for="item in typeList" :key="item.value" :label="item.value">
                {{ item.label }}
              </el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <p>PS：筛选出的方案仅展示前一百条</p>
        </el-col>
        <el-col :span="24">
          <el-form-item>
            <el-button @click="reset">重置</el-button>
            <el-button type="primary" @click="confirm">确认</el-button>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<script setup>
import {reactive, ref, defineProps, defineEmits, watch, computed} from 'vue'

const props = defineProps({
  formData: Object
})
const emit = defineEmits(['submit'])

let form = reactive(props.formData);

Object.assign(form, {
  choose: 1,
  checkedType: ['sky'],
  lucky: 100,
  landBase: 1,
  landMax: 5,
  skyMax: 3,
  landLucky: 1.3,
  skyLucky: 5.6
})
const ruleForm = ref(null)
const validatePass = (rule, value, callback) => {
  if (value === undefined) {
    callback(new Error('不能为空'))
  } else if (value === 0) {
    callback(new Error('不能为0'))
  } else {
    callback()
  }
}
const rules = reactive({
  lucky: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  landBase: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  skyBase: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  high: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  average: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  landMax: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  skyMax: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  landLucky: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  skyLucky: [
    {
      required: true,
      validator: validatePass,
      trigger: 'blur',
    },
  ],
  checkedType: [
    {
      required: true,
      message: '综合排序必选',
      trigger: 'change',
    },
  ],
})

const labelPosition = ref('top')

const luckyCalculate = () => {
  let list = [];
  let base, firstCycle, secondCycle, firstLucky, secondLucky;
  // 考虑到大小稀有切换，单次基础承天or纳地不一样，计算方式也不一样
  if (form.choose === 1) {
    base = form.landBase;
    firstCycle = form.landMax;
    secondCycle = form.skyMax;
    firstLucky = form.landLucky;
    secondLucky = form.skyLucky;

    for (let i = base; i <= firstCycle; i++) {
      // 纳地的幸运值
      let mLuck = (i - base) * firstLucky;
      let nLuckList = [];
      if (i - base !== 0) {
        for (let j = 0; j <= secondCycle; j++) {
          let nLuck = j * secondLucky;
          nLuckList.push({
            count: j, // 承天数量
            nLuck: nLuck, // 承天的幸运值
          });
        }
      }

      nLuckList.forEach(v => {
        const sum = parseFloat((mLuck + v.nLuck).toFixed(3));
        list.push({
          skyNumber: v.count, // 承天数量
          landNumber: i, // 纳地数量
          sum: sum, // 一手承天+纳地的幸运值
          high: parseFloat((form.high + sum).toFixed(3)), // 一手百炼的幸运值
          average: parseFloat((form.average + sum).toFixed(3)), // 一手普紫的幸运值
        })
      })
    }
  } else {
    base = form.skyBase;
    firstCycle = form.skyMax;
    secondCycle = form.landMax;
    firstLucky = form.skyLucky;
    secondLucky = form.landLucky;

    for (let i = base; i <= firstCycle; i++) {
      // 承天的幸运值
      let mLuck = (i - base) * firstLucky;
      let nLuckList = [];
      let jInit = i - base === 0 ? 0 : 1; // 不勾选使用承天纳地增长幸运值时判断
      for (let j = jInit; j <= secondCycle; j++) {
        let nLuck = j * secondLucky;
        nLuckList.push({
          count: j, // 纳地数量
          nLuck: nLuck, // 纳地的幸运值
        });
      }

      nLuckList.forEach(v => {
        const sum = parseFloat((mLuck + v.nLuck).toFixed(3));
        list.push({
          skyNumber: i, // 承天数量
          landNumber: v.count, // 纳地数量
          sum: sum, // 一手承天+纳地的幸运值
          high: parseFloat((form.high + sum).toFixed(3)), // 一手百炼的幸运值
          average: parseFloat((form.average + sum).toFixed(3)), // 一手普紫的幸运值
        })
      })
    }
  }
  const highSort = form.checkedType.indexOf('high') !== -1 ? 1 : 0;
  const skySort = form.checkedType.indexOf('sky') !== -1 ? 1 : 0;
  const landSort = form.checkedType.indexOf('land') !== -1 ? 1 : 0;


  // for (let i = base; i <= firstCycle; i++) {
  //   // 承天or纳地的幸运值
  //   let mLuck = (i - base) * firstLucky;
  //   let nLuckList = [];
  //   for (let j = 1; j <= secondCycle; j++) {
  //     let nLuck = j * secondLucky;
  //     nLuckList.push({
  //       count: j, // 纳地数量
  //       nLuck: nLuck, // 纳地or承天的幸运值
  //     });
  //   }
  //
  //   nLuckList.forEach(v => {
  //     const sum = parseFloat((mLuck + v.nLuck).toFixed(3));
  //     list.push({
  //       skyNumber: i, // 承天数量
  //       landNumber: v.count, // 纳地数量
  //       sum: sum, // 一手承天+纳地的幸运值
  //       high: parseFloat((form.high + sum).toFixed(3)), // 一手百炼的幸运值
  //       average: parseFloat((form.average + sum).toFixed(3)), // 一手普紫的幸运值
  //     })
  //   })
  // }


  let temp = [];
  const highList = list.map(v => v.high);
  // 计算出用最多的百炼次数
  const highMax = Math.ceil(form.lucky / Math.min(...highList));
  const averageList = list.map(v => v.average);
  // 计算出用最多的普紫次数
  const averageMax = Math.ceil(form.lucky / Math.min(...averageList));

  for (let i = 0; i <= highMax; i++) {
    for (let j = 0; j <= averageMax; j++) {
      list.forEach(v => {
            list.forEach(v1 => {
              const skyNumber = i * v.skyNumber + j * v1.skyNumber;
              const landNumber = i * v.landNumber + j * v1.landNumber;
              const optimal = i * highSort + skyNumber * skySort + (landNumber / 5) * landSort; // 综合排序
              // 遍历出幸运值大于form.lucky的方案
              if ((i * v.high + j * v1.average) >= form.lucky) {
                if (temp.length) {
                  const objIndex = temp.findIndex(v2 => v2.highCount === i && v2.averageCount === j);
                  // 剔除掉 大于form.lucky后的无效方案，比如100幸运，后续会有103、107、110等方案，实际上最有103才有价值
                  if (objIndex === -1) {
                    temp.push({
                      highCount: i,
                      averageCount: j,
                      skyNumber: skyNumber,
                      landNumber: landNumber,
                      highObj: v,
                      averageObj: v1,
                      optimal: optimal
                    })
                  }
                } else {
                  temp.push({
                    highCount: i, // 百炼装备数量
                    averageCount: j, // 普紫装备数量
                    skyNumber: skyNumber, // 承天数量
                    landNumber: landNumber, // 纳地数量
                    highObj: v, // 百炼搭配方案
                    averageObj: v1,// 普紫搭配方案
                    optimal: optimal, // 综合排序
                  })
                }
              }
            })
          }
      )
    }
  }

  return temp.sort((a, b) => a.optimal - b.optimal)


}

const typeList = [
  {value: 'high', label: '百炼'},
  {value: 'sky', label: '承天'},
  {value: 'land', label: '纳地'},
];

const changeData = () => {
  if (form.choose === 1) {
    form.checkedType = ['sky'];
    form.lucky = 100;
    form.landBase = 1;
    form.landMax = 5;
    form.skyMax = 3;
    form.landLucky = 1.3;
    form.skyLucky = 5.6;
  } else {
    form.checkedType = ['sky'];
    form.lucky = 100;
    form.skyBase = 2;
    form.landMax = 4;
    form.skyMax = 5;
    form.landLucky = 0.6;
    form.skyLucky = 2.6;
  }
}
const chooseChange = () => {
  changeData();
}
const reset = () => {
  Object.keys(form).forEach(v => {
    if (v !== 'choose')
      Object.assign(form, {
        [v]: null
      });
  })
  changeData();
  if (!ruleForm.value) return;
  ruleForm.value.resetFields();
}
const confirm = () => {
  if (!ruleForm.value) return;
  ruleForm.value.validate((valid) => {
    if (valid) {
      const list = luckyCalculate().slice(0, 100);
      emit('submit', list);
    }
  })

}

</script>

<style lang="scss" scoped>
.home-right {
  height: calc(100vh - 100px);
  overflow-y: auto;
  overflow-x: hidden;
}
</style>
