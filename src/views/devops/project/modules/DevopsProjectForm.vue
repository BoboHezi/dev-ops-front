<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span='24'>
            <a-form-item label="平台" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['projectCodeId', validatorRules.projectCodeId]" :trigger-change="true" dictCode="devops_code,code_name,code_name" placeholder="请选择平台" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="项目名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'projectName', validatorRules.projectName]" placeholder="请输入项目名称"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="描述" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectDesc']" placeholder="请输入描述"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="项目版本号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectVersionNumber']" placeholder="请输入项目版本号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="项目号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectNumber']" placeholder="请输入项目号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="客户号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectCustomNumber']" placeholder="请输入客户号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="渠道号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['projectChannelNumber']" placeholder="请输入渠道号"  ></a-input>
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
  import JTreeSelect from '@/components/jeecg/JTreeSelect'

  export default {
    name: 'DevopsProjectForm',
    components: {
      JTreeSelect
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
          projectCodeId: {
            rules: [
              { required: true, message: '请选择平台!'},
            ]
          },
          projectName: {
            rules: [
              { required: true, message: '请输入项目名称!'},
            ]
          },
          pid:{},
          name:{rules: [{ required: true, message: '请输入类型名称!' }]}
        },
        url: {
          add: "/project/devopsProject/add",
          edit: "/project/devopsProject/edit",
          queryById: "/project/devopsProject/queryById"
        },
        pidField:"pid"
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
          this.form.setFieldsValue(pick(this.model,'projectName','projectDesc','projectCodeId','projectVersionNumber','projectNumber','projectCustomNumber','projectChannelNumber'))
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
        this.form.setFieldsValue(pick(row,'projectName','projectDesc','projectCodeId','projectVersionNumber','projectNumber','projectCustomNumber','projectChannelNumber'))
      },
    }
  }
</script>