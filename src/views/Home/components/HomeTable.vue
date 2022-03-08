<template>
  <div class="home-center">
    <el-table :data="dataList" :height="520" style="width: 100%">
      <el-table-column type="index" width="50"/>
      <el-table-column type="expand">
        <template #default="props">
          <div style="width: 100%;overflow-x: hidden">
            <p>百炼装备方法:{{ highShow(props.row) }}</p>
            <el-row :gutter="20">
              <el-col :span="6">
                承天: {{ props.row.highObj.skyNumber }}
              </el-col>
              <el-col :span="6">
                纳地: {{ props.row.highObj.landNumber }}
              </el-col>
              <el-col :span="6">
                幸运值: {{ props.row.highObj.high }}
              </el-col>
            </el-row>
            <p>普紫方法: {{ averageShow(props.row) }}</p>
            <el-row :gutter="20">
              <el-col :span="6">
                承天: {{ props.row.averageObj.skyNumber }}
              </el-col>
              <el-col :span="6">
                纳地: {{ props.row.averageObj.landNumber }}
              </el-col>
              <el-col :span="6">
                幸运值: {{ props.row.averageObj.average }}
              </el-col>
            </el-row>
            <p>合计：{{ sumShow(props.row) }}</p>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="百炼装备数量" prop="highCount"/>
      <el-table-column label="普紫数量" prop="averageCount"/>
      <el-table-column label="纳地数量" prop="landNumber"/>
      <el-table-column label="承天数量" prop="skyNumber"/>
    </el-table>
  </div>
</template>

<script setup>
import {defineProps, reactive, ref, watch} from "vue";

const props = defineProps({
  list: {
    type: Array,
    default() {
      return []
    }
  }
})
let dataList = ref([]);
watch(() => props.list, () => {
  dataList.value = props.list;
})

const highShow = (item) => {
  return `${item.highCount}*${item.highObj.high}=${item.highCount * item.highObj.high} (百炼数量*幸运值)`;
};
const averageShow = (item) => {
  return `${item.averageCount}*${item.averageObj.average}=${item.averageCount * item.averageObj.average} (普紫数量*幸运值)`;
};
const sumShow = (item) => {
  return item.highCount * item.highObj.high + item.averageCount * item.averageObj.average;
};
</script>

<style lang="scss" scoped>
.home-center {
  height: calc(100vh - 100px);
}
</style>
