<template>
  <div class="home">
    <div class="home__header">
      一键计算器
    </div>
    <el-row :gutter="20">
      <el-col :span="12" :offset="3">
        <HomeTable v-model:list="list"></HomeTable>
      </el-col>
      <el-col :span="8" :offset="1">
        <Form v-model:formData="form" @submit="search"></Form>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import {reactive, ref} from 'vue';
import Form from './components/Form';
import HomeTable from './components/HomeTable';

let form = reactive({});
let list = ref([]);

const search = (val) => {
  list.value = [];
  list.value = val;
}

</script>
<style lang="scss" scoped>
.home {
  height: calc(100vh - 100px);
  padding: 20px;

  &__header {
    height: 60px;
    display: flex;
    justify-content: center;
    font-size: 24px;
  }
}
</style>
