<template>
  <a-spin :spinning='confirmLoading'>
    <j-form-container :disabled='formDisabled'>
      <a-form :form='form' slot='detail'>
        <a-row>
          <a-col :span='24'>
            <a-form-item label='名称' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-input v-decorator="['compileName', validatorRules.compileName]" placeholder='请输入名称'></a-input>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='描述' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-input v-decorator="['compileDesc']" placeholder='请输入描述'></a-input>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='编译平台' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-dict-select-tag type='list' v-decorator="['compilePlatformId', validatorRules.compilePlatformId]"
                                 :trigger-change='true' dictCode='devops_code,code_name,code_name'
                                 placeholder='请选择平台' />
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='新项目' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-input v-decorator="['newCompileProject',validatorRules.newCompileProject]"
                       placeholder='请输入新项目'></a-input>
            </a-form-item>
          </a-col>
          <!--          <a-col :span="24">-->
          <!--            <a-form-item label="项目" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
          <!--              <j-dict-select-tag type="list" v-decorator="['compileProjectId']" :trigger-change="true" dictCode="devops_project,project_name,project_name" placeholder="请选择项目" />-->
          <!--            </a-form-item>-->
          <!--          </a-col>-->
          <a-col :span='24'>
            <a-form-item label='版本类型' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-select v-decorator="['compileVariant' ,{  initialValue:'u' }]">
                <a-select-option value='u'>user</a-select-option>
                <a-select-option value='d'>userdebug</a-select-option>
                <a-select-option value='e'>eng</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='编译动作' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-select v-decorator="['compileAction',{  initialValue:'ota' }]">
                <a-select-option value='n'>new</a-select-option>
                <a-select-option value='ota'>ota</a-select-option>
                <a-select-option value='ota_factory'>ota_factory</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='是否签名' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-switch v-decorator="['compileIsSign',{  initialValue:'Y' }]"></j-switch>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='是否验收' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-switch v-decorator="['compileIsVerify',{  initialValue:'Y' }]"></j-switch>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='选择编译服务器' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-dict-select-tag type='list' v-decorator="['compileServerIp']" :trigger-change='true'
                                 dictCode='devops_server,server_ip,server_ip' placeholder='请选择你想编译的服务器，默认随机分配' />
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='邮箱通知抄送' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-input v-decorator="['compileSendEmail', validatorRules.compileSendEmail]"
                       placeholder='请输入邮箱通知抄送'></a-input>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='签名ftp账号' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-dict-select-tag type='list' v-decorator="['compileSignFtpId',validatorRules.compileSignFtpId]"
                                 :trigger-change='true' dictCode='devops_ftp,ftp_user_name,id'
                                 placeholder='请选择签名ftp账号'></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='优先级' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-select v-decorator="['compileQueueLevel' ,{  initialValue:'0' }]">
                <a-select-option value='0'>低</a-select-option>
                <a-select-option value='50'>中</a-select-option>
                <a-select-option value='100'>高</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <!--          <a-col :span="24">-->
          <!--            <a-form-item label="登录签名后台账号" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
          <!--              <j-dict-select-tag type="list" v-decorator="['compileLoginAccount']" :trigger-change="true" dictCode="devops_sign,sign_account,id" placeholder="请输入登录签名后台账号"  ></j-dict-select-tag>-->
          <!--            </a-form-item>-->
          <!--          </a-col>-->
          <a-col :span='24'>
            <a-form-item label='验收ftp账号' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-dict-select-tag v-decorator="['compileVerityFtpUserName']"
                                 :trigger-change='true' dictCode='devops_upload,upload_name,upload_name'
                                 placeholder='请输入验收ftp账号' />
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='签名平台' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <j-dict-select-tag type='list' v-decorator="['compileSvPlatformTerrace']" :trigger-change='true'
                                 dictCode='devops_sign_platform,platform_sign_product,platform_sign_product'
                                 placeholder='请输入签名平台'></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <a-col :span='24'>
            <a-form-item label='cherry pick' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-input v-for='(item,index) in letters' :key='index' v-model='letters[index]'
                       placeholder='请输入你要 cherry pick'></a-input>
            </a-form-item>
            <a-col :span='20' style='text-align: right'>
              <a-button @click='btnClickAdd'>添 加</a-button>
              <a-divider type='vertical' />
              <a-button @click='btnClickDelete'>删 除</a-button>
            </a-col>
          </a-col>
          <a-col v-if='showFlowSubmitButton' :span='24' style='text-align: center'>
            <a-button @click='submitForm'>提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

import { httpAction, getAction } from '@/api/manage'
import pick from 'lodash.pick'
import { validateDuplicateValue } from '@/utils/util'
import JTreeSelect from '@comp/jeecg/JTreeSelect'

export default {
  name: 'DevopsCompileForm',
  components: {
    JTreeSelect
  },
  props: {
    //流程表单data
    formData: {
      type: Object,
      default: () => {
      },
      required: false
    },
    //表单模式：true流程表单 false普通表单
    formBpm: {
      type: Boolean,
      default: false,
      required: false
    },
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data() {
    return {
      letters: [''],
      form: this.$form.createForm(this),
      treeValue: '',
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {
        compileName: {
          rules: [
            { required: true, message: '请输入名称!' }
          ]
        },
        compilePlatformId: {
          rules: [
            { required: true, message: '请输入平台!' }
          ]
        },
        newCompileProject: {
          rules: [
            { required: true, message: '请输入新项目!' }
          ]
        },
        compileProjectId: {
          rules: [
            { required: true, message: '请选择项目!' }
          ]
        },
        compileVariant: {
          rules: [
            { required: true, message: '请输入版本类型!' }
          ]
        },
        compileAction: {
          rules: [
            { required: true, message: '请输入编译动作!' }
          ]
        },
        compileIsSign: {
          rules: [
            { required: true, message: '请输入是否签名!' }
          ]
        },
        compileIsVerify: {
          rules: [
            { required: true, message: '请输入是否验收!' }
          ]
        },
        compileSendEmail: {
          rules: [
            { required: true, message: '请输入邮箱通知抄送!' }
          ]
        },
        compileSignFtpId: {
          rules: [
            { required: true, message: '请选择签名服务账号' }
          ]
        },
        pid: {}
      },
      url: {
        add: '/compile/devopsCompile/add',
        edit: '/compile/devopsCompile/edit',
        queryById: '/compile/devopsCompile/queryById'
      }
    }
  },
  computed: {
    formDisabled() {
      if (this.formBpm === true) {
        if (this.formData.disabled === false) {
          return false
        }
        return true
      }
      return this.disabled
    },
    showFlowSubmitButton() {
      if (this.formBpm === true) {
        if (this.formData.disabled === false) {
          return true
        }
      }
      return false
    }
  },
  created() {
    //如果是流程中表单，则需要加载流程表单data
    this.showFlowData()
  },
  methods: {
    btnClickAdd() {
      if (this.letters.length < 12)
        this.letters.push('')
      else {
        alert('最多添加12条cherry pick！')
      }
    },
    btnClickDelete() {
      if (this.letters.length > 1) {
        this.letters.pop()
      }
    },
    add() {
      this.edit({})
    },
    edit(record) {
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'compileName', 'compileBuildId', 'compileDesc', 'compilePlatformId', 'newCompileProject', 'compileProjectId', 'compileServerIp', 'compileVariant', 'compileAction', 'compileIsSign', 'compileIsVerify', 'compileStatus', 'compileLogUrl', 'compileSendEmail', 'compileBuildTime', 'compileSignFtpId', 'compileLoginAccount', 'compileVerityFtpUserName', 'compileSvPlatformTerrace', 'cherryPick','compileQueueLevel'))
        this.letters = this.model.cherryPick.split(';')
        if (this.letters.length > 1) {
          this.letters.pop()
        }
      })
    },
    //渲染流程表单数据
    showFlowData() {
      if (this.formBpm === true) {
        let params = { id: this.formData.dataId }
        getAction(this.url.queryById, params).then((res) => {
          if (res.success) {
            this.edit(res.result)
          }
        })
      }
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          if (this.letters[0] != '') {
            this.model.cherryPick = ''
            for (var i = 0; i < this.letters.length; i++) {
              if (this.letters[i] != '')
                this.model.cherryPick += this.letters[i] + ';'
            }
          }
          let formData = Object.assign(this.model, values)
          console.log('表单提交数据', formData)
          httpAction(httpurl, formData, method).then((res) => {
            if (res.success) {
              that.$message.success(res.message)
              that.$emit('ok')
            } else {
              that.$message.warning(res.message)
            }
          }).finally(() => {
            that.confirmLoading = false
          })
        }

      })
    },
    popupCallback(row) {
      this.form.setFieldsValue(pick(row, 'compileName', 'compileBuildId', 'compileDesc', 'compilePlatformId', 'newCompileProject', 'compileProjectId', 'compileServerIp', 'compileVariant', 'compileAction', 'compileIsSign', 'compileIsVerify', 'compileStatus', 'compileLogUrl', 'compileSendEmail', 'compileBuildTime', 'compileSignFtpId', 'compileLoginAccount', 'compileVerityFtpUserName', 'compileSvPlatformTerrace', 'cherryPick', 'compileQueueLevel'))
    }
  }
}
</script>