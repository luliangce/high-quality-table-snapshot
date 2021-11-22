<script setup lang="ts">
import { computed, reactive, ref, watchEffect, nextTick } from 'vue';
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
  height: 500,
  headerStyle: "background-color: black; \ncolor: white",
  stripe: true,
})

const headerStyle = computed(() => {
  const style: { [key: string]: string } = {}
  const lines = config.headerStyle.split(";")
  lines.forEach(line => {
    const kv = line.split(":")
    style[kv[0].trim()] = kv[1].trim();
  })
  console.log(style)
  return style
})


watchEffect(() => {
  if (!renderer.value) return
  renderer.value.style.width = config.width + "px"
  renderer.value.style.height = config.height + "px"

})
const downloadURL = ref("");


const capture = () => {
  if (!renderer.value) return
  const tableStyle = window.getComputedStyle(renderer.value.children[0])
  config.height = parseFloat(tableStyle.height)
  if (!renderer.value) return;
  nextTick(() => { //等待高度调整完毕
    if (!renderer.value) return
    html2canvas(renderer.value, {
      scale: scale.value,
    }).then(canvas => {
      downloadURL.value = canvas.toDataURL("image/png")
    });
  })

}

const tableData = computed(() => {
  const data: Record<string, string>[] = []
  const lines = text.value.split('\n')
  const headers = lines.shift()?.split(/(?:\t| )+/) as string[]
  lines.forEach(line => {
    const row: Record<string, string> = {}
    const words = line.split(/(?:\t| )+/)
    for (let i = 0; i < words.length; i++) {
      const word = words[i];
      row[headers[i]] = word
    }
    data.push(row)
  })
  return {
    headers,
    data
  }
})

</script>

<template>
  <el-row :gutter="30">
    <el-col :span="12">
      <el-input type="textarea" v-model="text" rows="20"></el-input>
    </el-col>
    <el-col :span="12">
      <el-form>
        <el-form-item>
          <el-form :inline="true">
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
        </el-form-item>
        <el-form-item>
          <el-link :href="downloadURL" download="1.png">下载</el-link>
        </el-form-item>
          </el-form>
        </el-form-item>
       <el-form-item label="stripe">
         <el-checkbox v-model="config.stripe"></el-checkbox>
       </el-form-item>
        <el-form-item label="标题样式" width="30%"> 
          <el-input type="textarea"   v-model="config.headerStyle"></el-input>
        </el-form-item>
      </el-form>
    </el-col>
  </el-row>
  <div v-if="downloadURL" style="position: absolute;right: 30px;bottom: 30px;width: 30%;">
    <h3>预览图片</h3>
    <div>
      <img :src="downloadURL" style="width: 100%;" />
    </div>
  </div>
  <div id="renderer" ref="renderer" style="display: block">
    <el-table
      :data="tableData.data"
      size="mini"
      :stripe="config.stripe"
      fit
      :border="true"
      :header-cell-style="() => headerStyle"
    >
      <el-table-column
        v-for="(header,i) in tableData.headers"
        resizable
        align="center"
        :key="i"
        :prop="header"
        :label="header"
      ></el-table-column>
    </el-table>
  </div>
</template>


<style lang="less">


</style>
