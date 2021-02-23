<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="服务器IP">
              <j-input placeholder="输入服务器IP模糊查询" v-model="queryParam.serverIp"></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a hidden @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button hidden type="primary" icon="download" @click="handleExportXls('服务器表单')">导出</a-button>
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
          <a @click="handleDetail(record)">详情</a>
          <a-dropdown hidden>
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
        <!-- 状态渲染模板 -->
        <template slot="customRenderStatus" slot-scope="serverStatus">
          <a-tag v-if="serverStatus==0" color="green">未使用</a-tag>
          <a-tag v-if="serverStatus==1" color="orange">已占用</a-tag>
          <a-tag v-if="serverStatus==2" color="blue">已关机</a-tag>
        </template>

      </a-table>
    </div>

    <devops-server-modal ref="modalForm" @ok="modalFormOk"></devops-server-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import DevopsServerModal from './modules/DevopsServerModal'

  export default {
    name: 'DevopsServerList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      DevopsServerModal
    },
    data () {
      return {
        description: '服务器表单管理页面',
        // 表头
        columns: [
          {
            title: '序号',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'IP地址',
            align:"center",
            dataIndex: 'serverIp'
          },
          {
            title:'主机名',
            align:"center",
            dataIndex: 'serverHost'
          },
          {
            title:'密码',
            align:"center",
            dataIndex: 'serverPassword'
          },
          {
            title:'状态信息',
            align:"center",
            dataIndex: 'serverStatus',
            scopedSlots: { customRender: 'customRenderStatus' },
            filterMultiple: false,
            filters: [
              { text: '未使用', value: '0' },
              { text: '已占用', value: '1' },
              { text: '已关机', value: '2' },
            ]
          },
          {
            title:'CPU使用率',
            align:"center",
            dataIndex: 'serverCpuUsageRate'
          },
          {
            title:'存储剩余空间',
            align:"center",
            dataIndex: 'serverAvailableStorage'
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
          list: "/server/devopsServer/list",
          delete: "/server/devopsServer/delete",
          deleteBatch: "/server/devopsServer/deleteBatch",
          exportXlsUrl: "/server/devopsServer/exportXls",
          importExcelUrl: "server/devopsServer/importExcel",
          
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
        fieldList.push({type:'string',value:'serverName',text:'名称',dictCode:''})
        fieldList.push({type:'string',value:'serverDesc',text:'描述',dictCode:''})
        fieldList.push({type:'string',value:'serverIp',text:'IP地址',dictCode:''})
        fieldList.push({type:'string',value:'serverHost',text:'主机名',dictCode:''})
        fieldList.push({type:'string',value:'serverPassword',text:'密码',dictCode:''})
        fieldList.push({type:'int',value:'serverStatus',text:'状态信息',dictCode:''})
        fieldList.push({type:'string',value:'serverCpuUsageRate',text:'CPU使用率',dictCode:''})
        fieldList.push({type:'string',value:'serverAvailableStorage',text:'存储剩余空间',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>