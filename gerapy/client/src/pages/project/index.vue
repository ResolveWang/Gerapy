<template>
  <div class="panel">
    <el-dialog :visible.sync="createProjectDialog" size="tiny">
      <el-form>
        <el-form-item label="项目名称">
          <el-input
            v-model="projectName" class="inline" placeholder="项目名称"
            size="small">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="createProjectDialog=false" size="small">取消</el-button>
        <el-button @click="onCreateProject()"
                   type="primary" size="small">添加
        </el-button>
      </div>
    </el-dialog>
    <panel-title title="项目管理">
      <!--<el-button @click.stop="onRefresh" size="mini">-->
      <!--<i class="fa fa-refresh"></i>-->
      <!--刷新-->
      <!--</el-button>-->
      <el-button type="primary" size="mini" @click="createProjectDialog=true">
        <i class="fa fa-plus"></i>
        创建
      </el-button>
    </panel-title>
    <div class="panel-body">
      <el-table
        :data="projects"
        v-loading="loadData"
        element-loading-text="拼命加载中"
        @selection-change="onBatchSelect"
        style="width: 100%;">
        <el-table-column
          align="center"
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          align="center"
          prop="name"
          label="项目名称"
          width="150">
        </el-table-column>
        <el-table-column
          align="center"
          label="版本描述"
          width="100">
          <template scope="props">
            <span v-if="buildInfos[props.row.name]">
              {{ buildInfos[props.row.name]['description'] }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="已打包"
          width="80">
          <template scope="props">
            <span v-if="buildInfos[props.row.name]">
              {{ buildInfos[props.row.name]['egg'] ? '是' : '否' }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="可配置"
          width="80">
          <template scope="props">
            <span v-if="buildInfos[props.row.name]">
              {{ buildInfos[props.row.name]['configurable'] ? '是' : '否' }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="打包名称"
          width="200">
          <template scope="props">
            <span v-if="buildInfos[props.row.name]">
              {{ buildInfos[props.row.name]['egg'] }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="打包时间"
          width="200">
          <template scope="props">
            <span v-if="buildInfos[props.row.name]">
              {{ buildInfos[props.row.name]['built_at'] }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="操作">
          <template scope="props">
            <router-link :to="{name: 'projectConfigure', params: {name: props.row.name}}" tag="span"
                         v-if="buildInfos[props.row.name]['configurable']">
              <el-button type="warning" size="mini">
                <i class="fa fa-edit"></i>
                配置
              </el-button>
            </router-link>
            <router-link :to="{name: 'projectEdit', params: {name: props.row.name}}" tag="span" v-else>
              <el-button type="warning" size="mini">
                <i class="fa fa-edit"></i>
                编辑
              </el-button>
            </router-link>
            <router-link :to="{name: 'projectDeploy', params: {name: props.row.name}}" tag="span">
              <el-button type="success" size="mini">
                <i class="fa fa-cloud-upload"></i>
                部署
              </el-button>
            </router-link>
            <!--<router-link :to="{name: 'projectMonitor', params: {name: props.row.name}}" tag="span">-->
            <!--<el-button type="info" size="mini">-->
            <!--<i class="fa fa-podcast"></i>-->
            <!--监控-->
            <!--</el-button>-->
            <!--</router-link>-->
            <el-button type="danger" size="mini" @click="onSingleDelete(props.row.name)">
              <i class="fa fa-remove"></i>
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <bottom-tool-bar>
        <el-button
          type="danger"
          icon="delete"
          size="mini"
          :disabled="batchSelect.length === 0"
          @click="onBatchDelete"
          slot="handler">
          <span>批量删除</span>
        </el-button>
      </bottom-tool-bar>
    </div>
  </div>

</template>
<script type="text/javascript">
  import {panelTitle, bottomToolBar} from 'components'
  export default{
    data(){
      return {
        createProjectDialog: false,
        projectName: null,
        projects: [],
        //请求时的loading效果
        loadData: false,
        //批量选择数组
        batchSelect: [],
        buildInfos: {}
      }
    },
    components: {
      panelTitle,
      bottomToolBar
    },
    created(){
      this.getProjectData()
    },
    methods: {
      onBatchSelect(val){
        this.batchSelect = val
      },
      getBuildInfo(name){
        this.loadData = true
        this.$fetch.apiProject.buildInfo({
          name: name
        }).then(({data: data}) => {
          this.$set(this.buildInfos, name, data)
          this.loadData = false
        }).catch(() => {
          this.loadData = false
        })
      },
      //获取数据
      getProjectData(){
        this.loadData = true
        this.$fetch.apiProject.projectList(
        ).then(({data: projects}) => {
          this.projects = projects
          this.loadData = false
          this.projects.forEach((project) => {
            console.log(project)
            this.getBuildInfo(project.name)
          })
        }).catch((data) => {
          console.log(data)
          this.loadData = false
        })
      },
      deleteProject(name) {
        this.loadData = true
        this.$fetch.apiProject.projectRemove({
          name: name
        }).then(() => {
          this.$message.success('删除成功')
          this.loadData = false
          this.getProjectData()
        }).catch((error) => {
          this.loadData = false
          this.$message.error('删除失败')
        })
      },
      // 单个删除
      onSingleDelete(name) {
        this.$confirm('此操作将删除项目相关所有数据, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.deleteProject(name)
        })
      },
      //批量删除
      onBatchDelete(){
        this.$confirm('此操作将批量删除选择数据, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.loadData = true
          console.log(this.batchSelect)
          this.batchSelect.forEach(({name: name}) => {
            this.deleteProject(name)
          })
        }).catch(() => {
          this.$message.error('批量删除出错')
        })
      },
      onCreateProject() {
        this.$fetch.apiProject.projectCreate({
          name: this.projectName
        }).then(() => {
          this.$message.success('创建成功')
          this.loadData = false
          this.$router.push({name: 'projectConfigure', params: {name: this.projectName}})
        }).catch((error) => {
          this.loadData = false
          this.$message.error('创建失败')
        })
      }
    }
  }
</script>
