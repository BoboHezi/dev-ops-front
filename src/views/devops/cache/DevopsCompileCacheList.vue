<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='jenkinsId'>
              <j-input placeholder='请输入jenkinsId' v-model='queryParam.jenkinsBuildId' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='项目名'>
              <j-input placeholder='请输入项目名' v-model='queryParam.projectName' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='邮件抄送'>
              <j-input placeholder='请输入邮件信息' v-model='queryParam.compileSendEmail' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='内部版本号'>
              <j-input placeholder='请输入内部版本号' v-model='queryParam.versionInternal' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <span style='float: left;overflow: hidden;' class='table-page-search-submitButtons'>
              <a-button type='primary' @click='searchQuery' icon='search'>查询</a-button>
              <a-button type='primary' @click='searchReset' icon='reload' style='margin-left: 8px'>重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class='table-operator' hidden>
      <a-button @click='handleAdd' type='primary' icon='plus'>新增</a-button>
      <a-button type='primary' icon='download' @click="handleExportXls('编译存放vmlinux')">导出</a-button>
      <a-upload name='file' :showUploadList='false' :multiple='false' :headers='tokenHeader' :action='importExcelUrl'
                @change='handleImportExcel'>
        <a-button type='primary' icon='import'>导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList='superFieldList' ref='superQueryModal' @handleSuperQuery='handleSuperQuery'
                     hidden></j-super-query>
      <a-dropdown v-if='selectedRowKeys.length > 0'>
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
          <a-dropdown>
            <a class='ant-dropdown-link'>更多 <a-icon type='down' /></a>
            <a-menu slot='overlay'>
              <a-menu-item>
                <a @click='handleDetail(record)'>详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title='确定删除吗?' @confirm='() => handleDelete(record.id)'>
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <devops-compile-cache-modal ref='modalForm' @ok='modalFormOk'></devops-compile-cache-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import DevopsCompileCacheModal from './modules/DevopsCompileCacheModal'

export default {
  name: 'DevopsCompileCacheList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    DevopsCompileCacheModal
  },
  data() {
    return {
      description: '编译存放vmlinux管理页面',
      // 表头
      columns: [
        {
          title: 'jenkinsID',
          align: 'center',
          dataIndex: 'jenkinsBuildId'
        },
        {
          title: '项目名',
          align: 'center',
          dataIndex: 'projectName'
        },
        {
          title: '抄送邮件',
          align: 'center',
          dataIndex: 'compileSendEmail'
        },
        {
          title: '平台',
          align: 'center',
          dataIndex: 'compilePlatformId'
        },
        {
          title: '内部版本号',
          align: 'center',
          dataIndex: 'versionInternal'
        },
        {
          title: '文件存放路径',
          align: 'center',
          dataIndex: 'cacheLocation'
        },
        // {
        //   title: '操作',
        //   dataIndex: 'action',
        //   align: 'center',
        //   fixed: 'right',
        //   width: 147,
        //   scopedSlots: { customRender: 'action' }
        // }
      ],
      url: {
        list: '/cache/devopsCompileCache/list',
        delete: '/cache/devopsCompileCache/delete',
        deleteBatch: '/cache/devopsCompileCache/deleteBatch',
        exportXlsUrl: '/cache/devopsCompileCache/exportXls',
        importExcelUrl: 'cache/devopsCompileCache/importExcel'

      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
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
      fieldList.push({ type: 'string', value: 'jenkinsBuildId', text: 'jenkinsID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'devopsCompileId', text: '编译任务ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'projectName', text: '项目名', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compileSendEmail', text: '抄送邮件', dictCode: '' })
      fieldList.push({ type: 'string', value: 'compilePlatformId', text: '平台', dictCode: '' })
      fieldList.push({ type: 'string', value: 'versionInternal', text: '内部版本号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'cacheLocation', text: '文件存放路径', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>