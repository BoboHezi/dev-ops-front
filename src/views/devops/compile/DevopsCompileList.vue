<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button hidden type="primary" icon="download" @click="handleExportXls('编译管理')">导出</a-button>
      <a-upload hidden name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button hidden type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query hidden :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown hidden v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div hidden class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <devops-compile-modal ref="modalForm" @ok="modalFormOk"></devops-compile-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import DevopsCompileModal from './modules/DevopsCompileModal'

  export default {
    name: 'DevopsCompileList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      DevopsCompileModal
    },
    data () {
      return {
        description: '编译管理管理页面',
        // 表头
        columns: [
          {
            title: 'TaskId',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'名称',
            align:"center",
            dataIndex: 'compileName'
          },
          {
            title:'任务ID',
            align:"center",
            dataIndex: 'compileBuildId'
          },
          {
            title:'描述',
            align:"center",
            dataIndex: 'compileDesc'
          },
          {
            title:'项目，平台，版本号',
            align:"center",
            dataIndex: 'compileProjectId'
          },
          {
            title:'版本类型',
            align:"center",
            dataIndex: 'compileVariant'
          },
          {
            title:'编译动作',
            align:"center",
            dataIndex: 'compileAction'
          },
          {
            title:'是否签名',
            align:"center",
            dataIndex: 'compileIsSign'
          },
          {
            title:'是否验收',
            align:"center",
            dataIndex: 'compileIsVerify'
          },
          {
            title:'任务状态',
            align:"center",
            dataIndex: 'compileStatus'
          },
          {
            title:'编译日志',
            align:"center",
            dataIndex: 'compileLogUrl'
          },
          {
            title:'邮箱通知抄送',
            align:"center",
            dataIndex: 'compileSendEmail'
          },
          {
            title:'编译开始时间',
            align:"center",
            dataIndex: 'compileBuildTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/compile/devopsCompile/list",
          delete: "/compile/devopsCompile/delete",
          deleteBatch: "/compile/devopsCompile/deleteBatch",
          exportXlsUrl: "/compile/devopsCompile/exportXls",
          importExcelUrl: "compile/devopsCompile/importExcel",
          
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'compileName',text:'名称',dictCode:''})
        fieldList.push({type:'int',value:'compileBuildId',text:'任务ID',dictCode:''})
        fieldList.push({type:'string',value:'compileDesc',text:'描述',dictCode:''})
        fieldList.push({type:'string',value:'compileProjectId',text:'项目，平台，版本号',dictCode:''})
        fieldList.push({type:'string',value:'compileVariant',text:'版本类型',dictCode:''})
        fieldList.push({type:'string',value:'compileAction',text:'编译动作',dictCode:''})
        fieldList.push({type:'int',value:'compileIsSign',text:'是否签名',dictCode:''})
        fieldList.push({type:'int',value:'compileIsVerify',text:'是否验收',dictCode:''})
        fieldList.push({type:'int',value:'compileStatus',text:'任务状态',dictCode:''})
        fieldList.push({type:'string',value:'compileLogUrl',text:'编译日志',dictCode:''})
        fieldList.push({type:'string',value:'compileSendEmail',text:'邮箱通知抄送',dictCode:''})
        fieldList.push({type:'date',value:'compileBuildTime',text:'编译开始时间'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>