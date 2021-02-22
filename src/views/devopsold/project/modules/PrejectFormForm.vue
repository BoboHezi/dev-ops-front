<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="项目平台" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['projectPlatform', validatorRules.projectPlatform]" :trigger-change="true" dictCode="platform_form,platform_name,platform_name" placeholder="请选择项目平台" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="项目名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectName', validatorRules.projectName]" placeholder="请输入项目名称"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="服务器 IP 地址 " :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['serverIpaddress', validatorRules.serverIpaddress]" placeholder="请输入服务器 IP 地址 "  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="版本类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['projectVariant', validatorRules.projectVariant]" :trigger-change="true" dictCode="devops_build,build_variant,build_variant_code" placeholder="请选择版本类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="编译签名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-switch v-decorator="['projectBuildSign']"  ></j-switch>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="编译动作" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['projectBuildAction', validatorRules.projectBuildAction]" :trigger-change="true" dictCode="devops_build,build_action,build_action_code" placeholder="请选择编译动作" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="项目路径" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectDir', validatorRules.projectDir]" placeholder="请输入项目路径"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="服务器主机名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['serverHost', validatorRules.serverHost]" placeholder="请输入服务器主机名"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="服务器密码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['serverPassword', validatorRules.serverPassword]" placeholder="请输入服务器密码"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
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

  export default {
    name: 'PrejectFormForm',
    components: {
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
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
    data () {
      return {
        form: this.$form.createForm(this),
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
          projectPlatform: {
            rules: [
              { required: true, message: '请输入项目平台!'},
            ]
          },
          projectName: {
            rules: [
              { required: true, message: '请输入项目名称!'},
            ]
          },
          serverIpaddress: {
            rules: [
              { required: true, message: '请输入服务器 IP 地址 !'},
            ]
          },
          projectVariant: {
            rules: [
              { required: true, message: '请输入版本类型!'},
            ]
          },
          projectBuildAction: {
            rules: [
              { required: true, message: '请输入编译动作!'},
            ]
          },
        },
        url: {
          add: "/devopsold/prejectForm/add",
          edit: "/devopsold/prejectForm/edit",
          queryById: "/devopsold/prejectForm/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'projectPlatform','projectName','serverIpaddress','projectVariant','projectBuildSign','projectBuildAction','projectStatus','projectDir','serverHost','serverPassword'))
        })
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'projectPlatform','projectName','serverIpaddress','projectVariant','projectBuildSign','projectBuildAction','projectStatus','projectDir','serverHost','serverPassword'))
      },
    }
  }
</script>
