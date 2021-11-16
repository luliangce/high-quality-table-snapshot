<script setup lang="ts">
import { computed, reactive, ref, watchEffect } from 'vue';
import html2canvas from 'html2canvas';
const renderer = ref<HTMLElement>()
const scale = ref(5)
const text = ref(`长形式 短形式 含义
-host -h 主机名
-user -u 用户名
-password -p 密码
-version -V 版本信息`);
const config = reactive({
  width: 500,
  height: 500
})
watchEffect(() => {
  if (!renderer.value) return
  renderer.value.style.width = config.width + "px"
  renderer.value.style.height = config.height + "px"

})
const downloadURL = ref("");


const capture = () => {
  if (!renderer.value) return;
  html2canvas(renderer.value, {
    scale: scale.value,
  }).then(canvas => {
    downloadURL.value = canvas.toDataURL("image/png")
  });
}
const headers = ref<string[]>([])
const tableData = computed(() => {
  const data: Record<string, string>[] = []
  const lines = text.value.split('\n')
  headers.value = lines.shift()?.split(/(?:\t| )+/) as string[]

  lines.forEach(line => {
    const row: Record<string, string> = {}
    const words = line.split(/(?:\t| )+/)
    for (let i = 0; i < words.length; i++) {
      const word = words[i];
      row[headers.value[i]] = word
    }
    data.push(row)
  })
  return data
})

</script>

<template>
  <el-row :gutter="30">
    <el-col :span="12">
      <el-input type="textarea" v-model="text" rows="20"></el-input>
    </el-col>
    <el-col :span="12">
      <el-form style="width: 50%;">
        <el-form-item label="scale">
          <el-input-number v-model="scale"></el-input-number>
        </el-form-item>
        <el-form-item label="宽度">
          <el-input-number v-model="config.width"></el-input-number>
        </el-form-item>
        <el-form-item label="高度">
          <el-input-number v-model="config.height"></el-input-number>
        </el-form-item>
        <el-form-item>
          <el-button @click="capture">渲染</el-button>
          <el-link :href="downloadURL" download="1.png">下载</el-link>
        </el-form-item>
        <el-form-item>
          <img :src="downloadURL" style="width: 100%;" />
        </el-form-item>
      </el-form>
    </el-col>
  </el-row>
 
  <div ref="renderer" style="display: block">
    <el-table
      :data="tableData"
      size="mini"
      stripe
      fit
      :border="true"
      :header-cell-style="{ backgroundColor: 'black', color: 'white' }"
    >
      <el-table-column
        v-for="(header,i) in headers"
        resizable
        align="center"
        :key="i"
        :prop="header"
        :label="header"
      ></el-table-column>
    </el-table>
  </div>
</template>



