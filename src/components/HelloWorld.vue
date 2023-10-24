<template>
  <div class="container">
    <button @click="openMessageBox">
      打开messagebox
    </button>
    <button @click="open(1); open(2)">
      打开两个弹框
    </button>
  </div>
  <TemplatePromise v-slot="{ resolve, args, isResolving }">
    <el-dialog :modelValue="true" :title="args[0]">
      <div>Dialog {{ args[1] }}</div>
      <p>可以打开控制台查看logs</p>
      <div class="flex gap-2 justify-end">
        <el-button @click="resolve('cancel')">
          取消
        </el-button>
        <el-button type="primary" :disabled="isResolving" @click="resolve(asyncFn())">
          {{ isResolving ? 'loading...' : '确认' }}
        </el-button>
      </div>
    </el-dialog>
  </TemplatePromise>
</template>

<script lang="jsx" setup>
import { reactive, ref } from 'vue';
import { createTemplatePromise } from '@vueuse/core'
import { ElDialog, ElMessageBox, ElForm, ElFormItem, ElInput, ElButton } from 'element-plus'

const TemplatePromise = createTemplatePromise()
const formRef = ref(null)
const form = reactive({ height: '', width: '' })
const rules = reactive({
  height: {
    required: true,
    trigger: 'blur'
  },
  width: {
    required: true,
    trigger: 'blur'
  }
})

function openMessageBox() {
  ElMessageBox({
    title: 'Message',
    showCancelButton: true,
    // message如果不是函数形式 绑定ref会失败
    message: () =>
      <ElForm
        ref={formRef}
        model={form}
        rules={rules}
      >
        <ElFormItem label="height" prop="height">
          <ElInput v-model={form.height}></ElInput>
        </ElFormItem>
        <ElFormItem label="width" prop="width">
          <ElInput v-model={form.width}></ElInput>
        </ElFormItem>
      </ElForm>
    ,
    beforeClose: (action, instance, done) => {
      console.log(action, instance)
      formRef.value && formRef.value.validate(status => {
        console.log('校验状态: ', status)
        if (status) done()
      })
    }
  })
}

async function open(idx) {
  console.log(idx, 'Before')
  const result = await TemplatePromise.start('Title', `Hello ${idx}`)
  console.log(idx, 'After', result)
}

function asyncFn() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('ok')
    }, 1000)
  })
}
</script>

<style></style>