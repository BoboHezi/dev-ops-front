<template>
  <div>
    <li class='title'>{{ newCompileProject }}</li>
    <a-card :bordered='false'>
      <li class='title'>名称： {{ compileName }}</li>
      <a class='title'>编译任务id： {{ compileBuildId }}</a>
      <li class='title'>编译描述： {{ compileDesc }}</li>
      <a class='title'>编译平台： {{ compilePlatformId }}</a>
      <li class='title'>编译项目名： {{ newCompileProject }}</li>
      <a class='title'>编译服务器IP： {{ compileServerIp }}</a>
      <div>
        <li class='title' v-if='compileVariant=="u"'>编译类型： user</li>
        <li class='title' v-else-if='compileVariant=="d"'>编译类型： user debug</li>
        <li class='title' v-else>编译类型： eng</li>
      </div>
      <a class='title'>编译动作： {{ compileAction }}</a>
      <div>
        <li class='title' v-if='compileIsSign=="Y"'>是否签名： 是</li>
        <li class='title' v-else>是否签名： 否</li>
      </div>
      <div>
        <a class='title' v-if='compileSaveVmlinux=="Y"'>是否保存VmLinux： 是</a>
        <a class='title' v-else>是否保存VmLinux： 否</a>
      </div>
      <div>
        <li class='title' v-if='compileIsVerify=="Y"'>是否验收： 是</li>
        <li class='title' v-else>是否验收： 否</li>
      </div>
      <div>
        <a v-if='compileStatus==-1' color='blue'>编译状态：   排队中</a>
        <a v-else-if='compileStatus==0' color='green'>编译状态：   编译成功</a>
        <a v-else-if='compileStatus==1' color='orange'>编译状态：   初始化</a>
        <a v-else-if='compileStatus==2' color='blue'>编译状态：   连接中</a>
        <a v-else-if='compileStatus==3' color='red'>编译状态：   参数错误</a>
        <a v-else-if='compileStatus==4' color='red'>编译状态：   新项目名错误</a>
        <a v-else-if='compileStatus==5' color='blue'>编译状态：   编译中</a>
        <a v-else-if='compileStatus==6' color='red'>编译状态：   编译失败</a>
        <a v-else-if='compileStatus==7' color='red'>编译状态：   编译停止</a>
        <a v-else-if='compileStatus==8' color='red'>编译状态：   没有代码</a>
        <a v-else-if='compileStatus==9' color='red'>编译状态：   cherry_pick失败</a>
        <a v-else-if='compileStatus==10' color='red'>编译状态：   上传失败</a>
        <a v-else-if='compileStatus==11' color='red'>编译状态：   提交签名失败</a>
        <a v-else-if='compileStatus==12' color='blue'>编译状态：   预处理中</a>
        <a v-else-if='compileStatus==13' color='red'>编译状态：   环境错误</a>
        <a v-else-if='compileStatus==14' color='red'>编译状态：   停止编译中</a>
        <a v-else color='red'>编译状态：   未知</a>
      </div>
      <li class='title'>编译的log路径： {{ compileLogUrl }}</li>
      <a class='title'>发送的邮件地址： {{ compileSendEmail }}</a>
      <li class='title'>编译开始的时间： {{ compileBuildTime }}</li>
      <a class='title'>编译签名的ftp： {{ compileSignFtpAccount }}</a>
      <li class='title'>编译验收的ftp： {{ compileVerityFtpUserName }}</li>
      <a class='title'>编译平台： {{ compileSvPlatformTerrace }}</a>
      <div>
        <li class='title' v-if='compileQueueLevel>=150'>编译等级：   加急</li>
        <li class='title' v-else-if='compileQueueLevel>=100'>编译等级：   高</li>
        <li class='title' v-else-if='compileQueueLevel>=50'>编译等级：   中</li>
        <li class='title' v-else>编译等级：   低</li>
      </div>
      <a class='title'>内部版本号： {{compileProjectNum}}</a>
    </a-card>
    <li class='title'>cherry pick:</li>
    <a-card :bordered='false'>
      <li v-for='le in letters'>{{ le }}</li>
    </a-card>
  </div>
</template>

<script>
import pick from 'lodash.pick'
import { getAction } from '@api/manage'
import JVxeTextareaCell from '@comp/jeecg/JVxeTable/components/cells/JVxeTextareaCell'

export default {
  components: { JVxeTextareaCell },
  data() {
    return {
      letters: [''], compileName: '',
      compileBuildId: '', compileDesc: '', compilePlatformId: '',
      newCompileProject: '', compileProjectId: '', compileServerIp: '',
      compileVariant: '', compileAction: '',
      compileIsSign: '', compileSaveVmlinux: '',
      compileIsVerify: '', compileStatus: '', compileLogUrl: '',
      compileSendEmail: '', compileBuildTime: '', compileSignFtpId: '',compileSignFtpAccount:'',
      compileLoginAccount: '', compileVerityFtpUserName: '',
      compileSvPlatformTerrace: '', compileQueueLevel: '',
      compileProjectNum: '',
      url: {
        queryById: '/compile/devopsCompile/queryById',
        queryByIdAccount: '/devops/devopsFtp/queryByIdAccount'
      }
    }
  },
  methods: {
    setDetail(record) {
      this.model = Object.assign({}, record)
      this.$nextTick(() => {
        this.compileName = this.model.compileName
        this.compileBuildId = this.model.compileBuildId
        this.compileDesc = this.model.compileDesc
        this.compilePlatformId = this.model.compilePlatformId
        this.newCompileProject = this.model.newCompileProject
        this.compileProjectId = this.model.compileProjectId
        this.compileServerIp = this.model.compileServerIp
        this.compileVariant = this.model.compileVariant
        this.compileAction = this.model.compileAction
        this.compileIsSign = this.model.compileIsSign
        this.compileSaveVmlinux = this.model.compileSaveVmlinux
        this.compileIsVerify = this.model.compileIsVerify
        this.compileStatus = this.model.compileStatus
        this.compileLogUrl = this.model.compileLogUrl
        this.compileSendEmail = this.model.compileSendEmail
        this.compileBuildTime = this.model.compileBuildTime
        this.compileSignFtpId = this.model.compileSignFtpId
        this.compileProjectNum = this.model.compileProjectNum
        this.getSignAccount()
        this.compileLoginAccount = this.model.compileLoginAccount
        this.compileVerityFtpUserName = this.model.compileVerityFtpUserName
        this.compileSvPlatformTerrace = this.model.compileSvPlatformTerrace
        this.compileQueueLevel = this.model.compileQueueLevel
        this.letters = this.model.cherryPick.split(';')
        if (this.letters.length > 1) {
          this.letters.pop()
        }
      })
    },
    getCompileDetail() {
      let params = { id: this.$route.query.id }
      getAction(this.url.queryById, params).then((res) => {
        if (res.success) {
          this.setDetail(res.result)
        }
      })
    },
    getSignAccount() {
      if (this.compileSignFtpId != null) {
        let params = { id: this.compileSignFtpId }
        getAction(this.url.queryByIdAccount, params).then((res) => {
          if (res.success) {
            this.compileSignFtpAccount = res.result
          }
        })
      }
    }

  },
  created() {
    this.getCompileDetail()
  },watch:{
    '$route'(to,from){
      this.getCompileDetail()
    }
  }
}
</script>
