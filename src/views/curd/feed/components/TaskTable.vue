<!--  -->
<template>
  <div class="task-table">
    <el-table
      :data="list"
      border
      style="width: 100%"
      @expand-change="expandChange"
      @cell-click="cellClick"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="name" label="名称" width="120" show-overflow-tooltip sortable></el-table-column>
      <el-table-column prop="description" label="描述" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="creat_time" label="创建时间" :show-overflow-tooltip="true" sortable>
        <template slot-scope="scope">
          <span>{{scope.row.creat_time | formatLongTime}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="update_time" label="更新时间" :show-overflow-tooltip="true" sortable>
        <template slot-scope="scope">
          <span>{{scope.row.update_time | formatLongTime}}</span>
        </template>
      </el-table-column>
      <el-table-column label="数据源数" label-class-name="ds_list_relation">
        <template slot-scope="scope">
          <span>{{ scope.row.ds_list_relation | count }}</span>
        </template>
      </el-table-column>
      <el-table-column label="模型数" label-class-name="model_list_relation">
        <template slot-scope="scope">
          <span>{{ scope.row.model_list_relation | count }}</span>
        </template>
      </el-table-column>
      <el-table-column label="标准数" label-class-name="standard_list_relation">
        <template slot-scope="scope">
          <span>{{ scope.row.standard_list_relation | count }}</span>
        </template>
      </el-table-column>
      <el-table-column label="主题数" label-class-name="subject_list">
        <template slot-scope="scope">
          <span>{{ scope.row.subject_list | count }}</span>
        </template>
      </el-table-column>
      <el-table-column label="规则数" label-class-name="rule_list">
        <template slot-scope="scope">
          <span>{{ scope.row.rule_list | count }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="320px">
        <template scope="scope">
          <el-button size="mini" title="编辑" type="primary" plain @click="handleEdit(scope.row)">
            <i class="fa fa-edit"></i>
          </el-button>
          <el-button
            size="mini"
            title="流程"
            type="primary"
            plain
            @click.stop="handleFeed(scope.row)"
          >
            <i class="fa fa-file-text"></i>
          </el-button>
          <el-tooltip class="item" effect="dark" content="关联" placement="bottom-end">
            <el-button type="success" plain size="mini" @click.stop="handleLink(scope.row)">
              <i class="fa fa-link"></i>
            </el-button>
          </el-tooltip>
          <el-button size="mini" title="删除" type="danger" plain @click="handleDel(scope.row)">
            <i class="fa fa-trash"></i>
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="流程列表" :visible.sync="dialogVisible" width="75%">
      <task-cont ref="taskCont" :taskInfo="taskInfo"></task-cont>
    </el-dialog>
    <!--  添加关联 -->
    <task-link ref="linkDialog" :config="dialog.linkConfig" @saveLink="saveLink"></task-link>
    <!-- 关联关系弹窗 -->
    <el-dialog :title="dialog.title" :visible.sync="dialog.realationVisible">
      <reladion-dialog :config="dialog.config"></reladion-dialog>
    </el-dialog>
  </div>
</template>

<script>
// 例如：import 《组件名称》 from '《组件路径》';
import ReladionDialog from '@/components/ReladionDialog/index.vue'
import TaskLink from '@/components/LinkDialog/TaskLink.vue'
import TaskCont from './TaskCont'
import TaskForm from './TaskForm'
export default {
  // import引入的组件需要注入到对象中才能使用/
  components: {
    TaskForm,
    TaskCont,
    ReladionDialog,
    TaskLink
  },
  props: ['list'],
  data () {
    // 这里存放数据
    return {
      filter: '',
      feedList: [],
      taskId: '',
      taskInfo: null,
      feedLoading: false,
      dialogVisible: false,
      dialog: {
        linkVisible: false,
        title: '关联关系',
        config: {
          data: [],
          fromCol: 'from_name',
          fromName: '',
          toCol: 'to_name',
          toName: ''
        },
        linkConfig: {
          selected: {
            depart: [],
            task: [],
            subject: [],
            business: [],
            standard: [],
            model: [],
            rule: [],
            datasource: []
          },
          type: 'SUBJECT',
          typeArr: ['depart', 'task']
        },
        realationVisible: false
      }
    };
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {
    filter (val) {
      console.log(val)
    }
  },
  filters: {
    count (arr) { // 计算个数
      if (Array.isArray(arr)) return arr.length;
      return 0
    }
  },
  // 方法集合
  methods: {
    getCont (id) {
      this.$refs.feed.getFeedList(id)
      // this.$refs.feedVersion.getList(id)
    },
    handleSelectionChange (val) {
      this.$emit('selectionChange', val)
    },
    handleLink (row) {
      console.log(row)
      this.dialog.linkConfig.selected = row
      this.$refs.linkDialog.dialog.visible = true
      /* this.$nextTick(() => {
        this.$refs.linkTable.clearSelection();
      }) */
    },
    // 保存关联
    saveLink () {
      this.$emit('refresh')
    },
    handleEdit (row) {
      this.$emit('addTask', { isEdit: true, data: row })
    },
    handleDel (row) {
      if (row.business_list.length > 0 || row.ds_list_relation.length > 0 || row.rule_list.length > 0 || row.standard_list_relation.length > 0 || row.subject_list.length > 0) {
        this.$message.warning('该任务有关联信息，无法删除')
        return;
      }
      this.$emit('delTask', row.id)
    },
    handleFeed (row) {
      this.taskInfo = row
      // this.dialogVisible = true
      this.$emit('handleFeed', row)
      // this.$refs.taskCont.getCont(row.id)
    },
    expandChange (row, expandedRows) {
      console.log(row)
      console.log(expandedRows)
      if (expandedRows.length === 0) return;
      this.taskInfo = row
      this.$refs.taskCont.getCont(row.id)
    },
    // 单元格被点击
    cellClick (row, column) {
      console.log(row)
      if (!column.labelClassName) return;
      this.dialog.config.fromName = '任务名称'
      this.dialog.config.fromCol = 'to_name'
      this.dialog.config.toCol = 'from_name'
      this.dialog.config.toName = column.label.slice(0, column.label.length - 1) + '名称'
      this.dialog.realationVisible = true;
      this.dialog.config.data = row[column.labelClassName]
      console.log('传递的值')
      console.log(this.dialog.config.data)
    }
  }
}
</script>
<style lang='scss' scoped>
//@import url(); 引入公共css类
</style>
