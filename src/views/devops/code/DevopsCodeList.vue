<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='平台名称'>
              <j-dict-select-tag placeholder='请选择平台名称' v-model='queryParam.codeName'
                                 dictCode='devops_code,code_name,code_name' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='服务器ip'>
              <j-dict-select-tag placeholder='请选择服务器ip' v-model='queryParam.codeServerId'
                                 dictCode='devops_server,server_ip,id' />
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
          <
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class='table-operator'>
      <a-button @click='handleAdd' type='primary' icon='plus'>新增</a-button>
      <a-button hidden type='primary' icon='download' @click="handleExportXls('源代码管理')">导出</a-button>
      <a-upload hidden name='file' :showUploadList='false' :multiple='false' :headers='tokenHeader'
                :action='importExcelUrl' @change='handleImportExcel'>
        <a-button hidden type='primary' icon='import'>导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query hidden :fieldList='superFieldList' ref='superQueryModal'
                     @handleSuperQuery='handleSuperQuery'></j-super-query>
      <a-dropdown v-if='selectedRowKeys.length > 0'>
        <a-menu slot='overlay'>
          <a-menu-item key='1' @click='batchDel'>
            <a-icon type='delete' />
            删除
          </a-menu-item>
        </a-menu>
        <a-button hidden style='margin-left: 8px'> 批量操作
          <a-icon type='down' />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div hidden class='ant-alert ant-alert-info' style='margin-bottom: 16px;'>
        <i class='anticon anticon-info-circle ant-alert-icon'></i> 已选择 <a
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

        <span slot='action' slot-scope='text, record'>
          <a @click='handleEdit(record)'>编辑</a>
          <a-divider type='vertical' />
          <a>
            <a @click='handleSync(record)' v-if='record.codeStatus==-1'>重新同步</a>
            <a @click='handleSync(record)' v-if='record.codeStatus==0'>同步代码</a>
            <a v-if='record.codeStatus==1'>正在同步</a>
            <a v-if='record.codeStatus==2'>同步完成</a>
          </a>
          <a-divider type='vertical' />
          <a>
            <a-popconfirm title='确定删除吗?' @confirm='() => handleDelete(record.id)'>
              <a>删除</a>
            </a-popconfirm>
          </a>
        </span>

        <!-- 状态渲染模板 -->
        <template slot='customRenderStatus' slot-scope='codeStatus'>
          <a-tag v-if='codeStatus==-1' color='red'>同步失败</a-tag>
          <a-tag v-if='codeStatus==0' color='orange'>未初始</a-tag>
          <a-tag v-if='codeStatus==1' color='green'>已启动</a-tag>
          <a-tag v-if='codeStatus==2' color='blue'>已成功</a-tag>
        </template>

      </a-table>
    </div>

    <devops-code-modal ref='modalForm' @ok='modalFormOk'></devops-code-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DevopsCodeModal from './modules/DevopsCodeModal'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { getAction } from '@api/manage'

export default {
  name: 'DevopsCodeList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    DevopsCodeModal
  },
  data() {
    return {
      description: '源代码管理管理页面',
      // 表头
      columns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '代码平台名称',
          align: 'center',
          dataIndex: 'codeName'
        },
        {
          title: 'repo路径',
          align: 'center',
          dataIndex: 'codeRepoUrl'
        },
        {
          title: '服务器IP',
          align: 'center',
          dataIndex: 'codeServerId_dictText'
        },
        {
          title: '拉取状态',
          align: 'center',
          dataIndex: 'codeStatus',
          scopedSlots: { customRender: 'customRenderStatus' },
          filterMultiple: false,
          filters: [
            { text: '同步失败', value: '-1' },
            { text: '未初始', value: '0' },
            { text: '已启动', value: '1' },
            { text: '已成功', value: '2' }
          ]
        },
        {
          title: '存放路径',
          align: 'center',
          dataIndex: 'codeDir'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 200,
          scopedSlots: { customRender: 'action' },
          key: 'actionKey'
        }
      ],
      url: {
        list: '/code/devopsCode/list',
        delete: '/code/devopsCode/delete',
        deleteBatch: '/code/devopsCode/deleteBatch',
        exportXlsUrl: '/code/devopsCode/exportXls',
        importExcelUrl: 'code/devopsCode/importExcel',
        syncCode: 'code/devopsCode/syncCode'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
    if (!this.checkPermission()) {
      this.columns = this.columns.filter(col => col.key != 'actionKey')
    }
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
      fieldList.push({ type: 'string', value: 'codeName', text: '名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'codeDesc', text: '描述', dictCode: '' })
      fieldList.push({ type: 'string', value: 'codeRepoUrl', text: 'repo路径', dictCode: '' })
      fieldList.push({ type: 'int', value: 'codeStatus', text: '拉取状态', dictCode: '' })
      fieldList.push({
        type: 'string',
        value: 'codeServerId',
        text: '服务器IP对应id',
        dictCode: 'devops_server,server_ip,id'
      })
      fieldList.push({ type: 'string', value: 'codeDir', text: '存放路径', dictCode: '' })
      this.superFieldList = fieldList
    },
    handleSync(record) {
      const that = this
      if (this.checkPermission()) {
        that.$confirm({
          title: '是否完成了同步操作',
          content: '是请按确认，不是轻按取消',
          onOk() {
            let params = { id: record.id }
            getAction(that.url.syncCode, params)
            that.loadData()
          }
        })
      }
    },
    checkPermission() {
      return this.tokenName == 'admin' || this.tokenName == 'imp01' || this.tokenName == 'imp02'
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>