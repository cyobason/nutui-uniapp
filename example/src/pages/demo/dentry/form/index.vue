<script setup lang="ts">
import type { FormInst, FormItemRule, FormItemRuleWithoutValidator } from 'nutui-uniapp'
import { reactive, ref } from 'vue'

/* eslint-disable no-console */
const formData = reactive({
  name: '',
  age: '',
  tel: '',
  address: '',
})
const basicData = reactive({
  name: '',
  age: '',
  time: '',
  address: '',
})
const dynamicRefForm = ref<FormInst | null>(null)
const dynamicForm = {
  state: reactive({
    name: '',
    time: [{
      key: 1,
      value: '',
    }],
  }),

  methods: {
    submit() {
      dynamicRefForm.value?.validate().then(({ valid, errors }) => {
        if (valid)
          console.log('success', dynamicForm)
        else
          console.log('error submit!!', errors)
      })
    },
    reset() {
      dynamicRefForm.value?.reset()
    },
    remove() {
      dynamicForm.state.time.splice(dynamicForm.state.time.length - 1, 1)
    },
    add() {
      dynamicForm.state.time.push({
        key: Date.now(),
        value: '',
      })
    },
  },
}

const formData2 = reactive({
  switch: false,
  checkbox: false,
  radio: 0,
  number: 0,
  rate: 3,
  range: 30,
  address: '',
  defaultFileList: [
    {
      name: '文件1.png',
      url: 'https://m.360buyimg.com/babel/jfs/t1/164410/22/25162/93384/616eac6cE6c711350/0cac53c1b82e1b05.gif',
      status: 'success',
      message: '上传成功',
      type: 'image',
    },
    {
      name: '文件2.png',
      url: 'https://m.360buyimg.com/babel/jfs/t1/164410/22/25162/93384/616eac6cE6c711350/0cac53c1b82e1b05.gif',
      status: 'uploading',
      message: '上传中...',
      type: 'image',
    },
  ],
})

const addressModule = reactive({
  state: {
    show: false,
    province: [
      { id: 1, name: '北京' },
      { id: 2, name: '广西' },
      { id: 3, name: '江西' },
      { id: 4, name: '四川' },
    ],
    city: [
      { id: 7, name: '朝阳区' },
      { id: 8, name: '崇文区' },
      { id: 9, name: '昌平区' },
      { id: 6, name: '石景山区' },
    ],
    country: [
      { id: 3, name: '八里庄街道' },
      { id: 9, name: '北苑' },
      { id: 4, name: '常营乡' },
    ],
    town: [],
  },
  methods: {
    show() {
      addressModule.state.show = !addressModule.state.show
      if (addressModule.state.show)
        formData2.address = ''
    },
    onChange({ _custom, next, value }: any) {
      formData2.address += value?.name
      const name = (addressModule as any).state[next]
      if (name.length < 1)
        addressModule.state.show = false
    },
  },
})

const ruleForm = ref<FormInst | null>(null)

function submit() {
  ruleForm.value?.validate().then(({ valid, errors }) => {
    if (valid)
      console.log('success', formData)
    else
      console.log('error submit!!', errors)
  })
}
function reset() {
  ruleForm.value?.reset()
}
// 失去焦点校验
function customBlurValidate(prop: string) {
  ruleForm.value?.validate(prop).then(({ valid, errors }) => {
    if (valid)
      console.log('success', formData)
    else
      console.log('error submit!!', errors)
  })
}
// 函数校验
const customValidator = async (val: string) => /^\d+$/.test(val)
async function customRulePropValidator(val: string, rule: FormItemRuleWithoutValidator) {
  return (rule?.reg as RegExp).test(val)
}
const nameLengthValidator = async (val: string) => val?.length >= 2
// Promise 异步校验
async function asyncValidator(val: string): Promise<string> {
  const telReg = /^400(-?)\d{7}$|^1\d{10}$|^0\d{2,3}-\d{7,8}$/
  return new Promise((resolve, reject) => {
    uni.showLoading({
      title: '模拟异步验证中...',
    })
    setTimeout(() => {
      uni.hideLoading()
      if (!val)
        // eslint-disable-next-line prefer-promise-reject-errors
        reject('请输入联系电话')

      else if (!telReg.test(val))
        // eslint-disable-next-line prefer-promise-reject-errors
        reject('联系电话格式不正确')

      else
        resolve('')
    }, 1000)
  })
}
</script>

<template>
  <div class="demo full">
    <h2 class="title">
      基础用法
    </h2>
    <nut-form>
      <nut-form-item label="昵称">
        <nut-input
          v-model="basicData.name"
          custom-class="nut-input-text"
          placeholder="请输入昵称"
          type="text"
        />
      </nut-form-item>
      <nut-form-item label="网龄">
        <nut-input
          v-model="basicData.age"
          custom-class="nut-input-text"
          placeholder="请输入网龄"
          type="text"
        />
      </nut-form-item>
      <nut-form-item label="时间">
        <nut-input
          v-model="basicData.time"
          custom-class="nut-input-text"
          placeholder="请输入时间"
          type="text"
        />
      </nut-form-item>
      <nut-form-item label="地址">
        <nut-input
          v-model="basicData.address"
          custom-class="nut-input-text"
          placeholder="请输入地址"
          type="text"
        />
      </nut-form-item>
      <nut-form-item label="备注">
        <nut-textarea placeholder="请输入备注" type="text" />
      </nut-form-item>
    </nut-form>
    <h2 class="title">
      动态表单
    </h2>
    <nut-form ref="dynamicRefForm" :model-value="dynamicForm.state">
      <nut-form-item
        label="昵称"
        prop="name"
        required
        :rules="[{ required: true, message: '请填写姓名' }]"
      >
        <nut-input
          v-model="dynamicForm.state.name"
          custom-class="nut-input-text"
          placeholder="请输入姓名"
          type="text"
        />
      </nut-form-item>
      <nut-form-item
        v-for="(item, index) in dynamicForm.state.time"
        :key="item.key"
        :label="`时间${index}`"
        :prop="`tels.${index}.value`"
        required
        :rules="[{ required: true, message: `请填写时间${index}` }]"
      >
        <nut-input
          v-model="item.value"
          custom-class="nut-input-text"
          :placeholder="`请输入时间${index}`"
          type="text"
        />
      </nut-form-item>
      <nut-cell>
        <nut-button size="small" custom-style="margin-right: 10px" @click="dynamicForm.methods.add">
          添加
        </nut-button>
        <nut-button size="small" custom-style="margin-right: 10px" @click="dynamicForm.methods.remove">
          删除
        </nut-button>
        <nut-button
          type="primary"
          custom-style="margin-right: 10px"
          size="small"
          @click="dynamicForm.methods.submit"
        >
          提交
        </nut-button>
        <nut-button size="small" @click="dynamicForm.methods.reset">
          重置提示状态
        </nut-button>
      </nut-cell>
    </nut-form>
    <h2 class="title">
      表单校验
    </h2>
    <nut-form
      ref="ruleForm"
      :model-value="formData"
      :rules="{
        name: [
          { required: true, message: '请填写姓名' },
          {
            message: 'Name should be at least two characters',
            validator: nameLengthValidator,
          },
        ],
      }"
    >
      <nut-form-item label="姓名" prop="name">
        <nut-input
          v-model="formData.name"
          class="nut-input-text"
          placeholder="请输入姓名，blur 事件校验"
          type="text"
          @blur="customBlurValidate('name')"
        />
      </nut-form-item>
      <nut-form-item
        label="网龄"
        prop="age"
        required
        :rules="[
          { required: true, message: '请填写网龄' },
          { validator: customValidator, message: '必须输入数字' },
          { validator: customRulePropValidator, message: '必须输入数字', reg: /^\d+$/ },
          { regex: /^(\d{1,2}|1\d{2}|200)$/, message: '必须输入0-200区间' },
        ] as FormItemRule[]"
      >
        <nut-input
          v-model="formData.age"
          custom-class="nut-input-text"
          placeholder="请输入网龄，必须数字且0-200区间"
          type="text"
        />
      </nut-form-item>
      <nut-form-item
        label="号码"
        prop="tel"
        required
        :rules="[
          { required: true, message: '请填写号码' },
          { validator: asyncValidator, message: '号码格式不正确' },
        ]"
      >
        <nut-input
          v-model="formData.tel"
          custom-class="nut-input-text"
          placeholder="请输入号码，异步号码格式"
          type="text"
        />
      </nut-form-item>
      <nut-form-item
        label="地址"
        prop="address"
        required
        :rules="[{ required: true, message: '请填写地址' }]"
      >
        <nut-input
          v-model="formData.address"
          custom-class="nut-input-text"
          placeholder="请输入地址"
          type="text"
        />
      </nut-form-item>
      <nut-cell>
        <nut-button
          type="primary"
          size="small"
          custom-style="margin-right: 10px"
          @click="submit"
        >
          提交
        </nut-button>
        <nut-button size="small" @click="reset">
          重置提示状态
        </nut-button>
      </nut-cell>
    </nut-form>
    <h2 class="title">
      表单类型
    </h2>
    <nut-form>
      <nut-form-item label="开关">
        <nut-switch v-model="formData2.switch" />
      </nut-form-item>
      <nut-form-item label="复选框">
        <nut-checkbox v-model="formData2.checkbox">
          复选框
        </nut-checkbox>
      </nut-form-item>
      <nut-form-item label="单选按钮">
        <nut-radio-group v-model="formData2.radio" direction="horizontal">
          <nut-radio label="1">
            选项1
          </nut-radio>
          <nut-radio disabled label="2">
            选项2
          </nut-radio>
          <nut-radio label="3">
            选项3
          </nut-radio>
        </nut-radio-group>
      </nut-form-item>
      <nut-form-item label="评分">
        <nut-rate v-model="formData2.rate" />
      </nut-form-item>
      <nut-form-item label="步进器">
        <nut-input-number v-model="formData2.number" />
      </nut-form-item>
      <nut-form-item label="滑块">
        <nut-range v-model="formData2.range" hidden-tag />
      </nut-form-item>
      <nut-form-item label="文件上传">
        <nut-uploader
          v-model:file-list="formData2.defaultFileList"
          url="http://服务地址"
          accept="image"
          maximum="3"
          multiple
        />
      </nut-form-item>
      <nut-form-item label="地址" is-link>
        <nut-input
          v-model="formData2.address"
          class="nut-input-text"
          readonly
          placeholder="请选择地址"
          type="text"
          @click="addressModule.methods.show"
        />
        <!-- nut-address -->
        <nut-address
          v-model:visible="addressModule.state.show"
          :province="addressModule.state.province"
          :city="addressModule.state.city"
          :country="addressModule.state.country"
          :town="addressModule.state.town"
          custom-address-title="请选择所在地区"
          @change="addressModule.methods.onChange"
        />
      </nut-form-item>
    </nut-form>

    <h2 class="title">
      自定义labe位置
    </h2>
    <nut-form label-position="top" star-position="right">
      <nut-form-item label="姓名" required>
        <nut-input
          v-model="basicData.name"
          class="nut-input-text"
          placeholder="请输入姓名"
          type="text"
        />
      </nut-form-item>
      <nut-form-item label="年龄" required>
        <nut-input
          v-model="basicData.age"
          class="nut-input-text"
          placeholder="请输入年龄"
          type="text"
        />
      </nut-form-item>
      <nut-form-item label-position="left" label="备注">
        <nut-textarea placeholder="请输入备注" type="text" />
      </nut-form-item>
    </nut-form>
  </div>
</template>

<style lang="scss" scoped></style>

<route lang="json">
{
  "style": {
    "navigationBarTitleText": "Form"
  }
}
</route>
