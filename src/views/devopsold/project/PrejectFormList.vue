<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="服务器IP">
              <j-dict-select-tag placeholder="请选择服务器IP" v-model="queryParam.serverIpaddress" dictCode="servers_form,servers_ip,servers_ip"/>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="平台名称">
              <j-dict-select-tag placeholder="请选择平台名称" v-model="queryParam.projectPlatform" dictCode="platform_form,platform_name,platform_name"/>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="项目名称">
              <j-input placeholder="输入项目名称模糊查询" v-model="queryParam.projectName"></j-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button hidden type="primary" icon="download" @click="handleExportXls('项目表单')">导出</a-button>
      <a-upload hidden name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button hidden type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query hidden :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown hidden v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button hidden style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>
    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;" hidden>
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
          <a @click="handleCompile(record)">开始编译</a>
          <a-divider type="vertical" />
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a @click="handleDetail(record)">详情</a>
        </span>

      </a-table>
    </div>

    <preject-form-modal ref="modalForm" @ok="modalFormOk"></preject-form-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PrejectFormModal from './modules/PrejectFormModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: 'PrejectFormList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      PrejectFormModal
    },
    data () {
      return {
        description: '项目表单管理页面',
        // 表头
        columns: [
          {
            title: '编号',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'项目平台',
            align:"center",
            dataIndex: 'projectPlatform'
          },
          {
            title:'项目名称',
            align:"center",
            dataIndex: 'projectName'
          },
          {
            title:'服务器 IP 地址 ',
            align:"center",
            dataIndex: 'serverIpaddress'
          },
          {
            title:'版本类型',
            align:"center",
            dataIndex: 'projectVariant'
          },
          {
            title:'编译签名',
            align:"center",
            dataIndex: 'projectBuildSign',
            customRender: (text) => (text ? filterMultiDictText(this.dictOptions['projectBuildSign'], text) : ''),
          },
          {
            title:'编译动作',
            align:"center",
            dataIndex: 'projectBuildAction'
          },
          {
            title:'任务状态',
            align:"center",
            dataIndex: 'projectStatus'
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
          list: "/devopsold/prejectForm/list",
          delete: "/devopsold/prejectForm/delete",
          deleteBatch: "/devopsold/prejectForm/deleteBatch",
          exportXlsUrl: "/devopsold/prejectForm/exportXls",
          importExcelUrl: "devopsold/prejectForm/importExcel",
          autoBuild: "/devopsold/build/autoBuild",
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
      this.$set(this.dictOptions, 'projectBuildSign', [{text:'是',value:'Y'},{text:'否',value:'N'}])
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
        fieldList.push({type:'string',value:'projectPlatform',text:'项目平台',dictCode:''})
        fieldList.push({type:'string',value:'projectName',text:'项目名称',dictCode:''})
        fieldList.push({type:'string',value:'serverIpaddress',text:'服务器 IP 地址 ',dictCode:''})
        fieldList.push({type:'string',value:'projectVariant',text:'版本类型',dictCode:''})
        fieldList.push({type:'switch',value:'projectBuildSign',text:'编译签名'})
        fieldList.push({type:'string',value:'projectBuildAction',text:'编译动作',dictCode:''})
        fieldList.push({type:'string',value:'projectStatus',text:'任务状态',dictCode:''})
        fieldList.push({type:'string',value:'projectDir',text:'项目路径',dictCode:''})
        fieldList.push({type:'string',value:'serverHost',text:'服务器主机名',dictCode:''})
        fieldList.push({type:'string',value:'serverPassword',text:'服务器密码',dictCode:''})
        this.superFieldList = fieldList
      },
      handleCompile(mrecord) {
        alert(mrecord.projectBuildAction)
        var vm = this
        vm.$http.post(
          this.url.autoBuild, {
            id: mrecord.id,
            projectPlatform: mrecord.projectPlatform,
            projectName: mrecord.projectName,
            serverIpaddress: mrecord.serverIpaddress,
            projectVariant: mrecord.projectVariant,
            projectBuildSign: mrecord.projectBuildSign,
            projectBuildAction: mrecord.projectBuildAction,
            projectStatus: mrecord.projectStatus,
            projectDir: mrecord.projectDir,
            serverHost: mrecord.serverHost,
            serverPassword: mrecord.serverPassword
          })
          .then(function(res) {
            if (res.success) {
              vm.$message.success(res.message)
              vm.$emit('ok')
            } else {
              vm.$message.warning(res.message)
            }
          })
          .catch(function(response) {
              console.log(response)
            }
          )
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>
