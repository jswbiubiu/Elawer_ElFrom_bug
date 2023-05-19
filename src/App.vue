<script lang="ts" setup>

import { onMounted, reactive, ref } from 'vue'
import { ElMessage } from 'element-plus'
import type { FormInstance } from 'element-plus'

import HelloWorld from './components/HelloWorld.vue';

const formRef = ref<FormInstance>()
const drawer = ref(false)

const loading = ref(false)
const type = ref<0 | 1>(0) // 0 新增 1 修改
const id = ref()
const formValue = reactive({
  address: '', // 地址
  detailAddress: '', // 详细地址
  postalCode: '', // 邮政编号
  linkMan: '', // 收货人
  linkMobile: '', // 收货电话
  activeAddress: 0 // 是否为默认地址，0: 不是， 1：是，默认为0
})




const changeAreaCascader = (obj) => {
  formValue.address = obj.pathLabels.join(',')
}

const mobileValidator = (rule, value, callback) => {
  if (!/1[3456789]\d{9}/.test(value)) {
    callback(new Error('请输入正确的手机号'))
  } else {
    callback()
  }
}

const codeValidator = (rule, value, callback) => {
  if (/\D/.test(value)) {
    callback(new Error('请输入正确的邮编'))
  } else {
    callback()
  }
}

const submitForm = () => {
  if (!formRef.value) return
  formRef.value.validate((valid) => {
    if (valid) {
      console.log('submit!')
      if (type.value === 0) handleAddAddress()
      if (type.value === 1) handleUpdateAddress()
    }
  })
}

const resetForm = () => {
  if (!formRef.value) return
  formRef.value.resetFields()
  console.log('重置', formRef.value, formValue)

  // // bug: 2023/05/18 测试发现 因为el-drawer组件使用的v-if来控制隐\显示 导致formRef的表单resetFields()重置方法清空有问题 才采用以下方法解决
  // if (!formValue.linkMan) {
  //   console.log('已清空 return')
  //   return
  // }
  // formValue.activeAddress = 0
  // formValue.address = ''
  // formValue.detailAddress = ''
  // formValue.linkMan = ''
  // formValue.linkMobile = ''
  // formValue.postalCode = ''
}

const handleAddAddress = () => {
  addAddress(formValue)
    .then(res => {
      if (res.state === 1) {
        drawer.value = false
        getAddressList()
        ElMessage({
          message: res.data || res.msg,
          type: 'success'
        })
      } else {
        ElMessage({
          message: res.data || res.msg,
          type: 'error'
        })
      }
    })
}

const handleUpdateAddress = () => {
  updateAddress({
    ...formValue,
    touristAddressId: id.value
  })
    .then(res => {
      if (res.state === 1) {
        drawer.value = false
        getAddressList()
        ElMessage({
          message: res.data || res.msg,
          type: 'success'
        })
      } else {
        ElMessage({
          message: res.data || res.msg,
          type: 'error'
        })
      }
    })
}

const clickDrawer = () => {
  type.value = 0
  drawer.value = true
}

const handleDrawerClosed = () => {
  formValue.activeAddress = 0
  formValue.address = ''
  formValue.detailAddress = ''
  formValue.linkMan = ''
  formValue.linkMobile = ''
  formValue.postalCode = ''
  id.value = ''
}

onMounted(() => {
})


</script>

<template>
  <HelloWorld msg="Vite + Vue" />
  <!-- <div>
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <HelloWorld msg="Vite + Vue" />
  <el-button @click="drawer = true">测试</el-button>
  
  <el-drawer
    v-model="drawer"
    title="新增收货地址"
    append-to-body
    @closed="handleDrawerClosed"
  >
    <el-form
      ref="formRef"
      class="login-main__form"
      label-position="right"
      label-width="100px"
      :model="formValue"
      style="max-width: 460px"
    >
      <el-form-item
        label="当前配送至"
      >
        <el-input
          value="中国大陆"
          placeholder="请输入收货人"
          disabled
        />
      </el-form-item>
      <el-form-item
        label="收货人"
        prop="linkMan"
        :rules="[
          { required: true, message: '联系人不能为空' }
        ]"
      >
        <el-input
          v-model="formValue.linkMan"
          placeholder="请输入收货人"
        />
      </el-form-item>

      <el-form-item
        label="手机号"
        prop="linkMobile"
        :rules="[
          { required: true, message: '手机号不能为空' },
          { validator: mobileValidator }
        ]"
      >
        <el-input
          v-model="formValue.linkMobile"
          type="tel"
          placeholder="请输入手机号"
        />
      </el-form-item>

      <el-form-item
        label="邮编"
        prop="postalCode"
        :rules="[
          { validator: codeValidator }
        ]"
      >
        <el-input
          v-model="formValue.postalCode"
          placeholder="请输入邮编"
        />
      </el-form-item>

      <el-form-item
        label="用户地址"
        prop="address"
        :rules="[
          { required: true, message: '地址不能为空' }
        ]"
      >
        <AreaCascader
          v-model="formValue.address"
          @change="changeAreaCascader"
        />
      </el-form-item>
      <el-form-item
        label="详细地址"
        prop="detailAddress"
        :rules="[
          { required: true, message: '详细地址不能为空' }
        ]"
      >
        <el-input
          v-model="formValue.detailAddress"
          type="textarea"
          placeholder="请输入详细地址"
        />
      </el-form-item>
      <el-form-item
        label=""
        prop="detailAddress"
      >
        <el-checkbox
          v-model="formValue.activeAddress"
          :true-label="1"
          :false-label="0"
        >
          设置为默认收货地址
        </el-checkbox>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          @click="submitForm"
        >
          提交
        </el-button>
        <el-button
          @click="resetForm"
        >
          重置
        </el-button>
      </el-form-item>
    </el-form>
  </el-drawer> -->

</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
