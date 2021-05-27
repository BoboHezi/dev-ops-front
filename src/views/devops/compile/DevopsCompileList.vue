<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='服务器IP'>
              <j-dict-select-tag placeholder='请选择服务器IP' v-model='queryParam.compileServerIp'
                                 dictCode='devops_server,server_ip,server_ip' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='平台名称'>
              <j-dict-select-tag placeholder='请选择平台名称' v-model='queryParam.compilePlatformId'
                                 dictCode='devops_code,code_name,code_name' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='项目名称'>
              <j-input placeholder='输入项目名称模糊查询' v-model='queryParam.compileProjectId'></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='新项目名称'>
              <j-input placeholder='输入项目名称模糊查询' v-model='queryParam.newCompileProject'></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <span style='float: left;overflow: hidden;' class='table-page-search-submitButtons'>
              <a-button type='primary' @click='searchQuery' icon='search'>查询</a-button>
              <a-button type='primary' @click='searchReset' icon='reload' style='margin-left: 8px'>重置</a-button>
              <a-button @click='handleAdd' type='primary' icon='plus' style='margin-left: 8px'>新增</a-button>
              <a hidden @click='handleToggleSearch' style='margin-left: 8px'>
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class='table-operator'>
      <a-button hidden type='primary' icon='download' @click="handleExportXls('编译管理')">导出</a-button>
      <a-upload hidden name='file' :showUploadList='false' :multiple='false' :headers='tokenHeader'
                :action='importExcelUrl' @change='handleImportExcel'>
        <a-button hidden type='primary' icon='import'>导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query hidden :fieldList='superFieldList' ref='superQueryModal'
                     @handleSuperQuery='handleSuperQuery'></j-super-query>
      <a-dropdown hidden v-if='selectedRowKeys.length > 0'>
        <a-menu slot='overlay'>
          <a-menu-item key='1' @click='batchDel'>
            <a-icon type='delete' />
            删除
          </a-menu-item>
        </a-menu>
        <a-button style='margin-left: 8px'> 批量操作
          <a-icon type='down' />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div hidden class='ant-alert ant-alert-info' style='margin-bottom: 16px;'>
        <i class='action action-info-circle ant-alert-icon'></i> 已选择 <a
        style='font-weight: 600'>{{ selectedRowKeys.length }}</a>项
        <a style='margin-left: 24px' @click='onClearSelected'>清空</a>
      </div>

      <a-table
        ref='table'
        size='middle'
        :scroll='{x:true}'
        bordered
        rowKey='id'
        :columns='columns'
        :dataSource='dataSource'
        :pagination='ipagination'
        :loading='loading'
        class='j-table-force-nowrap'
        @change='handleTableChange'>

        <template slot='htmlSlot' slot-scope='text'>
          <div v-html='text'></div>
        </template>
        <template slot='imgSlot' slot-scope='text'>
          <span v-if='!text' style='font-size: 12px;font-style: italic;'>无图片</span>
          <img v-else :src='getImgView(text)' height='25px' alt=''
               style='max-width:80px;font-size: 12px;font-style: italic;' />
        </template>
        <template slot='fileSlot' slot-scope='text'>
          <span v-if='!text' style='font-size: 12px;font-style: italic;'>无文件</span>
          <a-button
            v-else
            :ghost='true'
            type='primary'
            icon='download'
            size='small'
            @click='downloadFile(text)'>
            下载
          </a-button>
        </template>

        <span slot='checkLog' slot-scope='text, record'>
          <a @click='handleCheckLog(record)'
             v-if='record.compileLogUrl!=null&&record.compileLogUrl.toString().length!=0'>查看</a>
          <a v-if='record.compileLogUrl==null||record.compileLogUrl.toString().length==0'></a>
        </span>

        <span slot='action' slot-scope='text, record'>
          <a @click='cancelCompile(record)' v-if='record.compileStatus==-1'>取消排队</a>
          <a @click='handleCompile(record)' v-if='record.compileStatus==1'>开始编译</a>
          <a @click='handleStopCompile(record)' v-if='record.compileStatus==2'>停止编译</a>
          <a @click='handleCompile(record)' v-if='record.compileStatus==3'>重新编译</a>
          <a @click='handleCompile(record)' v-if='record.compileStatus==4'>重新编译</a>
          <a @click='handleStopCompile(record)' v-if='record.compileStatus==5'>停止编译</a>
          <a @click='handleCompile(record)' v-if='record.compileStatus==6'>重新编译</a>
          <a @click='handleCompile(record)' v-if='record.compileStatus==7'>重新编译</a>
          <a @click='handleCompile(record)' v-if='record.compileStatus==8'>重新编译</a>
          <a v-if='record.compileStatus==0'>编译完成</a>

          <a-divider type='vertical' />
          <a-dropdown>
            <a class='ant-dropdown-link'>更多 <a-icon type='down' /></a>
            <a-menu slot='overlay'>
              <a-menu-item>
                <a @click='handleEdit(record)'>编辑</a>
              </a-menu-item>
              <a-menu-item>
                <a @click='handleDetail(record)'>详情</a>
              </a-menu-item>
              <a-menu-item>
                <a @click='handleCopy(record)'>复制添加</a>
              </a-menu-item>

              <a-menu-item hidden>
                <a-popconfirm title='确定删除吗?' @confirm='() => handleDelete(record.id)'>
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
        <!-- 状态渲染模板 -->
        <template slot='customRenderStatus' slot-scope='compileStatus'>
          <a-tag v-if='compileStatus==-1' color='blue'>排队中</a-tag>
          <a-tag v-if='compileStatus==0' color='green'>编译成功</a-tag>
          <a-tag v-if='compileStatus==1' color='orange'>初始化</a-tag>
          <a-tag v-if='compileStatus==2' color='blue'>连接中</a-tag>
          <a-tag v-if='compileStatus==3' color='red'>参数错误</a-tag>
          <a-tag v-if='compileStatus==4' color='red'>新项目名错误</a-tag>
          <a-tag v-if='compileStatus==5' color='blue'>编译中</a-tag>
          <a-tag v-if='compileStatus==6' color='red'>编译失败</a-tag>
          <a-tag v-if='compileStatus==7' color='red'>编译停止</a-tag>
          <a-tag v-if='compileStatus==8' color='red'>没有代码</a-tag>
        </template>
      </a-table>
    </div>

    <devops-compile-modal ref='modalForm' @ok='modalFormOk'></devops-compile-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DevopsCompileModal from './modules/DevopsCompileModal'
import { filterMultiDictText } from '@comp/dict/JDictSelectUtil'
import AreaChartTy from '@comp/chart/AreaChartTy'
import { getAction } from '@api/manage'

export default {
  name: 'DevopsCompileList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    AreaChartTy,
    DevopsCompileModal
  },
  data() {
    return {
      description: '编译管理管理页面',
      // 表头
      columns: [
        {
          title: 'TaskId',
          dataIndex: 'compileBuildId',
          align: 'center'
        },
        {
          title: '新项目',
          align: 'center',
          dataIndex: 'newCompileProject'
        },
        // {
        //   title: '项目',
        //   align: 'center',
        //   dataIndex: 'compileProjectId'
        // },
        {
          title: '平台',
          align: 'center',
          dataIndex: 'compilePlatformId'
        },
        {
          title: '编译服务器ip',
          align: 'center',
          dataIndex: 'compileServerIp'
        },
        {
          title: '版本类型',
          align: 'center',
          dataIndex: 'compileVariant',
          customRender: function(text) {
            if (text == 'u') {
              return 'user'
            }
            if (text == 'd') {
              return 'userdebug'
            }
            if (text == 'e') {
              return 'eng'
            }
          }
        },
        {
          title: '编译动作',
          align: 'center',
          dataIndex: 'compileAction',
          scopedSlots: { customRender: 'customRenderActionStatus' },
          filterMultiple: false,
          customRender: function(text) {
            if (text == 'ota') {
              return 'ota'
            }
            if (text == 'n') {
              return 'new'
            }
          }
        },
        {
          title: '是否签名',
          align: 'center',
          dataIndex: 'compileIsSign',
          customRender: (text) => (text ? filterMultiDictText(this.dictOptions['compileIsSign'], text) : '')
        },
        {
          title: '是否验收',
          align: 'center',
          dataIndex: 'compileIsVerify',
          customRender: (text) => (text ? filterMultiDictText(this.dictOptions['compileIsVerify'], text) : '')
        },
        {
          title: '任务状态',
          align: 'center',
          dataIndex: 'compileStatus',
          scopedSlots: { customRender: 'customRenderStatus' },
          filterMultiple: false
        },
        {
          title: '邮箱通知抄送',
          align: 'center',
          dataIndex: 'compileSendEmail'
        },
        {
          title: '编译开始时间',
          align: 'center',
          dataIndex: 'compileBuildTime'
        },
        {
          title: '编译结束时间',
          align: 'center',
          dataIndex: 'compileBuildFinishTime'
        },
        {
          title: '签名ftp账号',
          align: 'center',
          dataIndex: 'compileSignFtpId_dictText'
        },
        // {
        //   title: '登录签名后台账号',
        //   align: 'center',
        //   dataIndex: 'compileLoginAccount_dictText'
        // },
        {
          title: '验收ftp账号',
          align: 'center',
          dataIndex: 'compileVerityFtpUserName'
        },
        {
          title: '签名验收平台',
          align: 'center',
          dataIndex: 'compileSvPlatformTerrace'
        },
        {
          title: '编译日志',
          dataIndex: 'checkLog',
          align: 'center',
          fixed: 'right',
          width: 100,
          scopedSlots: { customRender: 'checkLog' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/compile/devopsCompile/list',
        delete: '/compile/devopsCompile/delete',
        deleteBatch: '/compile/devopsCompile/deleteBatch',
        exportXlsUrl: '/compile/devopsCompile/exportXls',
        importExcelUrl: 'compile/devopsCompile/importExcel',
        autoCompile: '/compile/devopsCompile/autoCompile',
        stopCompile: '/compile/devopsCompile/stopCompile',
        handleCopy: '/compile/devopsCompile/handleCopy',
        cancelCompile: '/compile/devopsCompile/cancelCompile'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.$set(this.dictOptions, 'compileIsSign', [{ text: '是', value: 'Y' }, { text: '否', value: 'N' }])
    this.$set(this.dictOptions, 'compileIsVerify', [{ text: '是', value: 'Y' }, { text: '否', value: 'N' }])
    this.getSuperFieldList()
    this.timer = setInterval(() => {
      this.loadData()
    }, 1000 * 30)
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    initDictConfig() {
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'compileName', text: '名称', dictCode: '' })
      fieldList.push({ type: 'int', value: 'compileBuildId', text: '任务ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileDesc', text: '描述', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compilePlatformId', text: '平台', dictCode: '' })
      fieldList.push({ type: 'string', value: 'newCompileProject', text: '新项目名', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileProjectId', text: '项目，版本号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileServerIp', text: '编译服务器ip', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileVariant', text: '版本类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileAction', text: '编译动作', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileIsSign', text: '是否签名', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileIsVerify', text: '是否验收', dictCode: '' })
      fieldList.push({ type: 'int', value: 'compileStatus', text: '任务状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileLogUrl', text: '编译日志', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileSendEmail', text: '邮箱通知抄送', dictCode: '' })
      fieldList.push({ type: 'date', value: 'compileBuildTime', text: '编译开始时间' })
      fieldList.push({
        type: 'string',
        value: 'compileSignFtpId',
        text: '签名ftpID',
        dictCode: 'devops_ftp,ftp_user_name,id'
      })
      fieldList.push({
        type: 'string',
        value: 'compileLoginAccount',
        text: '登录签名账号',
        dictCode: 'devops_ftp,ftp_user_name,id'
      })
      fieldList.push({ type: 'string', value: 'compileVerityFtpUserName', text: '验收服务器账号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileSvPlatformTerrace', text: '签名平台', dictCode: '' })
      this.superFieldList = fieldList
    },
    handleCheckLog(mRecord) {
      window.open(mRecord.compileLogUrl)
    },
    handleCopy(mRecord) {
      const that = this
      this.$confirm({
        title: '复制项目',
        content: '项目名称：' + (mRecord.compileProjectId == null ?
          mRecord.newCompileProject : mRecord.compileProjectId),
        onOk() {
          that.$http.post(
            that.url.handleCopy, {
              id: mRecord.id
            })
          that.loadData()
        },
        onCancel() {
        }
      })
    },
    handleStopCompile(mRecord) {
      const that = this
      this.$confirm({
        title: '停止编译',
        content: '是否停止编译？',
        onOk() {
          that.$http.post(
            that.url.stopCompile, {
              id: mRecord.id,
              compileJenkinsJobName: mRecord.compileJenkinsJobName,
              compileJenkinsJobId: mRecord.compileJenkinsJobId
            })
            .then(function(res) {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$emit('ok')
                that.$message.warning(res.message)
              }
            })
            .catch(function(response) {
              that.$emit('发送失败')
                console.log(response)
              }
            )
        },
        onCancel() {
        }
      })
    },
    cancelCompile(mRecord) {
      const that = this
      this.$confirm({
        title: '取消排队',
        content: '是否取消排队,重置项目？',
        onOk() {
          that.$http.post(
            that.url.cancelCompile, {
              id: mRecord.id
            })
            .then(function(res) {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$emit('ok')
                that.$message.warning(res.message)
              }
            })
            .catch(function(response) {
              that.$emit('发送失败')
                console.log(response)
              }
            )
          that.loadData()
        },
        onCancel() {
        }
      })
    },
    handleCompile(mRecord) {
      const that = this
      this.$confirm({
        title: '编译项目',
        content: '项目名称：' + (mRecord.compileProjectId == null ? mRecord.newCompileProject
          : mRecord.compileProjectId),
        onOk() {
          that.$http.post(
            that.url.autoCompile, {
              id: mRecord.id,
              createBy: mRecord.createBy,
              compileBuildId: mRecord.compileBuildId,
              compileName: mRecord.compileName,
              compilePlatformId: mRecord.compilePlatformId,
              newCompileProject: mRecord.newCompileProject,
              compileProjectId: mRecord.compileProjectId,
              compileServerIp: mRecord.compileServerIp,
              compileVariant: mRecord.compileVariant,
              compileAction: mRecord.compileAction,
              compileIsSign: mRecord.compileIsSign,
              compileIsVerify: mRecord.compileIsVerify,
              compileStatus: mRecord.compileStatus,
              compileSendEmail: mRecord.compileSendEmail,
              compileSignFtpId: mRecord.compileSignFtpId,
              compileLoginAccount: mRecord.compileLoginAccount,
              compileVerityFtpUserName: mRecord.compileVerityFtpUserName,
              compileSvPlatformTerrace: mRecord.compileSvPlatformTerrace,
              cherryPick: mRecord.cherryPick
            }).then((res) => {
            if (res.success) {
              that.$message.success(res.message)
              that.loadData()
              that.$emit('ok')
            } else {
              that.loadData()
              that.$message.warning(res.message)
            }
          }).catch(function(response) {
              console.log(response)
            }
          )
        },
        onCancel() {
        }
      })
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>