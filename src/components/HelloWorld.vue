<script lang="ts" setup>
import { onMounted, reactive, ref, readonly } from 'vue'
// import AreaCascader, { CHANGE_VALUE } from '@/components/AreaCascader/index.vue'
// import { addAddress, addressList, activeAddress, removeAddress, updateAddress } from '@/api/tourist'
// import { ADDRESS_TYPE } from '@/api/type/tourist_type'
import { Plus } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import type { FormInstance } from 'element-plus'

const formRef = ref<FormInstance>()

const drawer = ref(false)
const loading = ref(false)
const type = ref<0 | 1>(0) // 0 新增 1 修改
const id = ref()
const formValue = ref({
  address: '', // 地址
  detailAddress: '', // 详细地址
  postalCode: '', // 邮政编号
  linkMan: '', // 收货人
  linkMobile: '' as string | number, // 收货电话
  activeAddress: 0 as 0 | 1 // 是否为默认地址，0: 不是， 1：是，默认为0
})

const tableData = ref<any[]>([])

// activeAddress: 0 | 1 // 是否为默认地址，0: 不是， 1：是，默认为0
// touristAddressId: number // ID
const tableTitle = readonly([
  {
    prop: 'linkMan',
    label: '收货人',
    width: '108px',
    align: 'center'
  },
  {
    prop: 'address',
    label: '所在地',
    width: '136px',
    align: 'center'
  },
  {
    prop: 'detailAddress',
    label: '详细地址',
    align: 'center'
  },
  {
    prop: 'postalCode',
    label: '邮政编号',
    width: '98px',
    align: 'center'
  },
  {
    prop: 'linkMobile',
    label: '电话/手机',
    width: '132px',
    align: 'center'
  }
])

const mobileValidator = (rule: any, value: any, callback: any) => {
  if (!/1[3456789]\d{9}/.test(value)) {
    callback(new Error('请输入正确的手机号'))
  } else {
    callback()
  }
}

const codeValidator = (rule: any, value: any, callback: any) => {
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
  console.log('重置', formRef.value, formValue.value)

  // bug: 2023/05/18 测试发现 因为el-drawer组件使用的v-if来控制隐\显示 导致formRef的表单resetFields()重置方法清空有问题 才采用以下方法解决
  // if (!formValue.value.linkMan && !formValue.value.detailAddress) {
  //   console.log('已清空 return')
  //   return
  // }
  // formValue.value.activeAddress = 0
  // formValue.value.address = ''
  // formValue.value.detailAddress = ''
  // formValue.value.linkMan = ''
  // formValue.value.linkMobile = ''
  // formValue.value.postalCode = ''
}

const handleAddAddress = () => {
  addAddress(formValue.value)
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
    ...formValue.value,
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

const clickEdit = (row: any) => {
  // drawer.value = false
  // setTimeout(() => {
  //   console.log('开');
    
  //   drawer.value = true
  //   formValue.value.activeAddress = row.activeAddress
  //   formValue.value.address = row.address
  //   formValue.value.detailAddress = row.detailAddress
  //   formValue.value.linkMan = row.linkMan
  //   formValue.value.linkMobile = row.linkMobile
  //   formValue.value.postalCode = row.postalCode
  //   id.value = row.touristAddressId
  //   type.value = 1
  // })
  
  type.value = 1
  console.log(row)
  const copyRow = JSON.parse(JSON.stringify(row))
  console.log(row == copyRow)
  console.log(row === copyRow)
  console.log(row, copyRow)
  
  drawer.value = true
  formValue.value = copyRow
  // formValue.value.activeAddress = copyRow.activeAddress
  // formValue.value.address = copyRow.address
  // formValue.value.detailAddress = copyRow.detailAddress
  // formValue.value.linkMan = copyRow.linkMan
  // formValue.value.linkMobile = copyRow.linkMobile
  // formValue.value.postalCode = copyRow.postalCode
  id.value = copyRow.touristAddressId
}

const clickDelete = (row: any) => {
  removeAddress({
    touristAddressId: row.touristAddressId as number
  }).then(res => {
    if (res.state === 1) {
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

const clicKDefault = (row: any) => {
  activeAddress({
    touristAddressId: row.touristAddressId as number
  }).then(res => {
    if (res.state === 1) {
      console.log(res)
      getAddressList()
      ElMessage({
        message: res.msg,
        type: 'success'
      })
    } else {
      ElMessage({
        message: res.msg,
        type: 'error'
      })
    }
  })
}

const getAddressList = () => {
  
  return tableData.value = [
    {
      touristAddressId: 56,
      address: "广东省,广州市,海珠区",
      detailAddress: "12",
      postalCode: "1",
      linkMan: "234",
      linkMobile: "13545454545",
      activeAddress: 0
    },
    {
      touristAddressId: 61,
      address: "广东省,广州市,海珠区",
      detailAddress: "123",
      postalCode: "123",
      linkMan: "测试2",
      linkMobile: "13652725969",
      activeAddress: 0
    },
    {
      touristAddressId: 62,
      address: "广东省,广州市,海珠区",
      detailAddress: "123",
      postalCode: "123",
      linkMan: "测试333",
      linkMobile: "13652725969",
      activeAddress: 1
    }
  ]
  addressList()
    .then(res => {
      console.log(res)
      if (res.state === 1) {
        tableData.value = res.data
      }
    })
}

const clickDrawer = () => {
  type.value = 0
  drawer.value = true
}

const handleDrawerClosed = () => {
  // if (!formRef.value) return
  // formRef.value.resetFields()
  console.log('关闭 ---  111', formRef.value, formValue.value)

  formValue.value.activeAddress = 0
  formValue.value.address = ''
  formValue.value.detailAddress = ''
  formValue.value.linkMan = ''
  formValue.value.linkMobile = ''
  formValue.value.postalCode = ''
  id.value = ''
  console.log('关闭 ---  222', formRef.value, formValue.value)
}

onMounted(() => {
  getAddressList()
})

</script>

<template>
  <div class="user-address">
    <div class="space-between flex-align mb20 user-address__top">
      <div class="user-address__title ">
        我的收货地址
        <span>最多可新建10个地址</span>
      </div>
      <el-button
        v-if="tableData.length < 10"
        :icon="Plus"
        type="primary"
        plain
        @click="clickDrawer"
      >
        新增收货地址
      </el-button>
    </div>
    <el-table
      :data="tableData"
      :header-cell-style="{
        background: '#F5F7FA'
      }"
      style="width: 100%"
      border
    >
      <el-table-column
        v-for="(item, index) in tableTitle"
        :key="index"
        v-bind="{...item}"
      />
      <el-table-column
        label="操作"
        align="center"
        width="220px"
      >
        <template #default="scope">
          <el-button
            link
            type="primary"
            @click="clickEdit(scope.row)"
          >
            修改
          </el-button>
          <el-popconfirm
            title="确认删除?"
            @confirm="clickDelete(scope.row)"
          >
            <template #reference>
              <el-button
                link
                color="#666666"
                plain
              >
                删除
              </el-button>
            </template>
          </el-popconfirm>
          <el-popconfirm
            v-if="(scope.row.activeAddress !== 1)"
            title="确认设为默认?"
            @confirm="clicKDefault(scope.row)"
          >
            <template #reference>
              <el-button
                link
                color="#303133"
                plain
              >
                设为默认
              </el-button>
            </template>
          </el-popconfirm>
          <el-button
            v-else
            size="small"
            color="#FF6100"
            plain
            disabled
          >
            当前默认
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
  <el-drawer
    v-model="drawer"
    v-loading="loading"
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
        <el-input
          v-model="formValue.address"
          placeholder="请输入用户地址"
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
  </el-drawer>
</template>

<style lang="scss" scoped>
.user-address {
  padding: 16px 20px;
  background-color: #fff;
  box-sizing: border-box;
  .user-address__top {
    padding: 4px 16px 4px 26px;
    background: #F8F9FA;
    box-sizing: border-box;
    .user-address__title {
      font-size: 18px;
      font-weight: 600;
      span {
        font-weight: 400;
        font-size: 14px;
        color: #999;
      }
    }
  }
}
</style>
