<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='原地址'>
              <j-input placeholder='输入原地址模糊查询' v-model='queryParam.otaOriginalDir'></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='目标地址'>
              <j-input placeholder='输入目标地址模糊查询' v-model='queryParam.otaNewDir'></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='邮件抄送'>
              <j-input placeholder='输入邮件抄送模糊查询' v-model='queryParam.otaEmail'></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <span style='float: left;overflow: hidden;' class='table-page-search-submitButtons'>
              <a-button type='primary' @click='searchQuery' icon='search'>查询</a-button>
              <a-button type='primary' @click='searchReset' icon='reload' style='margin-left: 8px'>重置</a-button>
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
      <a-button @click='handleAdd' type='primary' icon='plus'>新增</a-button>
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
        <span slot='action' slot-scope='text, record'>
          <a @click='cancelOta(record)' v-if='record.otaStatus==-1'>取消排队</a>
          <a @click='handleOta(record)' v-else-if='record.otaStatus==1'>制作ota</a>
          <a v-else-if='record.otaStatus==2'>制作中</a>
          <a v-else-if='record.otaStatus==0'>制作完成</a>
          <a @click='handleOta(record)' v-else>重新制作</a>
          <a-divider type='vertical' />

          <a-dropdown>
            <a class='ant-dropdown-link'>更多 <a-icon type='down' /></a>
            <a-menu slot='overlay'>
              <a-menu-item>
                <a @click='handleEdit(record)'>编辑</a>
              </a-menu-item>
              <a-menu-item hidden>
                <a @click='handleDetail(record)'>详情</a>
              </a-menu-item>
              <a-menu-item>
                <a @click='handleCopy(record)'>复制</a>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
        <!-- 状态渲染模板 -->
        <template slot='customOtaStatus' slot-scope='otaStatus'>
          <a-tag v-if='otaStatus==-1' color='blue'>排队中</a-tag>
          <a-tag v-else-if='otaStatus==0' color='green'>制作成功</a-tag>
          <a-tag v-else-if='otaStatus==1' color='orange'>初始化</a-tag>
          <a-tag v-else-if='otaStatus==2' color='green'>制作中</a-tag>
          <a-tag v-else-if='otaStatus==3' color='red'>制作失败</a-tag>
          <a-tag v-else-if='otaStatus==4' color='orange'>参数错误</a-tag>
          <a-tag v-else-if='otaStatus==5' color='red'>下载失败</a-tag>
          <a-tag v-else-if='otaStatus==6' color='red'>未知平台</a-tag>
          <a-tag v-else-if='otaStatus==7' color='red'>系统错误</a-tag>
          <a-tag v-else-if='otaStatus==8' color='red'>上传失败</a-tag>
          <a-tag v-else color='red'>未知</a-tag>
        </template>

        <!-- 字符串超长截取省略号显示-->
        <span slot='otaSuccessDir' slot-scope='text'>
          <j-ellipsis :value='text' :length='30' />
        </span>

        <!-- 字符串超长截取省略号显示-->
        <span slot='otaOriginalDir' slot-scope='text'>
          <j-ellipsis :value='text' :length='30' />
        </span>

        <!-- 字符串超长截取省略号显示-->
        <span slot='otaNewDir' slot-scope='text'>
          <j-ellipsis :value='text' :length='30' />
        </span>

        <span slot='checkLog' slot-scope='text, record'>
          <a @click='HCheckLog(record)' v-if='record.otaLogUrl!=null&&record.otaLogUrl.toString().length!=0'>查看</a>
          <a v-if='record.otaLogUrl==null||record.otaLogUrl.toString().length==0'></a>
        </span>

      </a-table>
    </div>

    <devops-diff-ota-modal ref='modalForm' @ok='modalFormOk' />
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DevopsDiffOtaModal from './modules/DevopsDiffOtaModal'
import { getAction } from '@api/manage'

export default {
  name: 'DevopsDiffOtaList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    DevopsDiffOtaModal
  },
  data() {
    return {
      description: '做差分包表格管理页面',
      // 表头
      columns: [
        {
          title: '任务ID',
          align: 'center',
          dataIndex: 'otaTaskId'
        },
        {
          title: 'ota平台',
          align: 'center',
          dataIndex: 'otaPlatform'
        },
        {
          title: '邮件',
          align: 'center',
          dataIndex: 'otaEmail'
        },
        {
          title: 'ota状态',
          align: 'center',
          dataIndex: 'otaStatus',
          scopedSlots: { customRender: 'customOtaStatus' }
        },
        {
          title: '上个项目号',
          align: 'center',
          dataIndex: 'otaOriginalDir',
          scopedSlots: { customRender: 'otaOriginalDir' }
        },
        {
          title: '目标项目号',
          align: 'center',
          dataIndex: 'otaNewDir',
          scopedSlots: { customRender: 'otaNewDir' }
        },
        {
          title: 'ota包',
          align: 'center',
          dataIndex: 'otaSuccessDir',
          scopedSlots: { customRender: 'otaSuccessDir' }
        },
        {
          title: 'OTA日志',
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
        list: '/ota/devopsDiffOta/list',
        delete: '/ota/devopsDiffOta/delete',
        deleteBatch: '/ota/devopsDiffOta/deleteBatch',
        exportXlsUrl: '/ota/devopsDiffOta/exportXls',
        importExcelUrl: 'ota/devopsDiffOta/importExcel',
        cancelOta: 'ota/devopsDiffOta/cancelOta',
        handleOta: 'ota/devopsDiffOta/handleOta',
        handleCopy: 'ota/devopsDiffOta/handleCopy'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  filters: {
    ellipsis(value) {
      if (!value) return ''
      if (value.length > 10) {
        return value.split('/')[value.split('/').length - 1].split('_signed_')[0]
      }
      return value
    }
  },
  created() {
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
      fieldList.push({ type: 'string', value: 'otaOriginalDir', text: '上个项目地址', dictCode: '' })
      fieldList.push({ type: 'string', value: 'otaNewDir', text: '目标项目地址', dictCode: '' })
      fieldList.push({ type: 'string', value: 'otaPlatform', text: 'ota平台', dictCode: '' })
      fieldList.push({ type: 'int', value: 'otaStatus', text: 'ota成功状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'otaEmail', text: '邮件', dictCode: '' })
      fieldList.push({ type: 'string', value: 'otaLogUrl', text: 'log地址', dictCode: '' })
      this.superFieldList = fieldList
    },
    HCheckLog(mRecord) {
      window.open(mRecord.otaLogUrl)
    },
    cancelOta(mRecord) {
      const that = this
      that.$confirm({
        title: '取消ota制作',
        content: '是否取消ota排队中？',
        onOk() {
          let params = { id: mRecord.id }
          getAction(that.url.cancelOta, params).then((res) => {
            if (res.success) {
              that.$message.success(res.message)
              that.loadData()
            } else {
              that.$message.warning(res.message)
            }
          })
        },
        onCancel() {
        }
      })
    },
    handleCopy(mRecord) {
      const that = this
      if (that.tokenName == 'admin' || this.tokenName == 'imp01' || this.tokenName == 'imp02' || that.tokenName == mRecord.createBy) {
        that.$confirm({
          title: '复制ota',
          content: '你要复制任务ID为: ' + mRecord.otaTaskId + '项目',
          onOk() {
            let params = { id: mRecord.id }
            getAction(that.url.handleCopy, params).then((res) => {
              if (res.success) {
                that.$message.success(res.message)
                that.loadData()
              } else {
                that.$message.warning(res.message)
              }
            })
          },
          onCancel() {
          }
        })
      } else {
        alert('你没有该权限！')
      }
    },
    handleOta(mRecord) {
      const that = this
      that.$confirm({
        title: '制作ota',
        content: '确认ota制作?',
        onOk() {
          let params = { id: mRecord.id }
          getAction(that.url.handleOta, params).then((res) => {
            if (res.success) {
              that.$message.success(res.message)
              that.loadData()
            } else {
              that.$message.warning(res.message)
            }
          })
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