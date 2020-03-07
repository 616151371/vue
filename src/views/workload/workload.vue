<template>
  <div class="app-container">
    <div class="filter-container">
      <el-form>
        <el-form-item>
          <el-button type="primary" icon="plus" @click="showCreate" v-if="hasPerm('workload:add')">添加
          </el-button>
        </el-form-item>
      </el-form>
    </div>
    <el-table :data="list" v-loading.body="listLoading" element-loading-text="拼命加载中" border fit
              highlight-current-row>
      <el-table-column align="center" label="序号" width="80">
        <template slot-scope="scope">
          <span v-text="getIndex(scope.$index)"> </span>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="type" label="工作量类型" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="hours" label="工时" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="content" label="内容" style="width: 60px;"></el-table-column>
      <el-table-column align="center" label="创建时间" width="170">
        <template slot-scope="scope">
          <span>{{scope.row.createTime}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="管理" width="200" v-if="hasPerm('workload:update') || hasPerm('workload:delete')">
        <template slot-scope="scope">
          <el-button type="primary" icon="edit" @click="showUpdate(scope.$index)">修改</el-button>
          <el-button type="primary" icon="edit" @click="showDelete(scope.$index)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="listQuery.pageNum"
      :page-size="listQuery.pageRow"
      :total="totalCount"
      :page-sizes="[10, 20, 50, 100]"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form class="small-space" :model="tempworkload" label-position="left" label-width="60px"
               style='width: 300px; margin-left:50px;'>
        <el-form-item label="类型">
          <el-select v-model="tempworkload.type" placeholder="请选择">
            <el-option
              value="基础工作">
            </el-option>
            <el-option
              value="临时安排">
            </el-option>
            <el-option
              value="调课/加班">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="类型">
          <el-select v-model="tempworkload.hours" placeholder="请选择">
            <el-option
              value="1小时">
            </el-option>
            <el-option
              value="2小时">
            </el-option>
            <el-option
              value="3小时">
            </el-option>
            <el-option
              value="4小时">
            </el-option>
            <el-option
              value="5小时">
            </el-option>
            <el-option
              value="6小时">
            </el-option>
            <el-option
              value="7小时">
            </el-option>
            <el-option
              value="8小时">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="内容">
          <el-input type="text" v-model="tempworkload.content">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="createworkload">创 建</el-button>
        <el-button v-if="dialogStatus=='update'" type="primary"  @click="updateworkload">修 改</el-button>
        <el-button v-if="dialogStatus=='delete'" v-else type="primary" v-else @click="deleteworkload">删 除</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        totalCount: 0, //分页组件--数据总条数
        list: [],//表格的数据
        listLoading: false,//数据加载等待动画
        listQuery: {
          pageNum: 1,//页码
          pageRow: 50,//每页条数
          name: ''
        },
        dialogStatus: 'create',
        dialogFormVisible: false,
        textMap: {
          update: '编辑',
          create: '创建'
        },
        tempworkload: {
          id: "",
          type:"",
          hours:"",
          content: ""
        }
      }
    },
    created() {
      this.getList();
    },
    methods: {
      getList() {
        //查询列表
        if (!this.hasPerm('workload:list')) {
          return
        }
        this.listLoading = true;
        this.api({
          url: "/workload/listworkload",
          method: "get",
          params: this.listQuery
        }).then(data => {
          this.listLoading = false;
          this.list = data.list;
          this.totalCount = data.totalCount;
        })
      },
      handleSizeChange(val) {
        //改变每页数量
        this.listQuery.pageRow = val
        this.handleFilter();
      },
      handleCurrentChange(val) {
        //改变页码
        this.listQuery.pageNum = val
        this.getList();
      },
      getIndex($index) {
        //表格序号
        return (this.listQuery.pageNum - 1) * this.listQuery.pageRow + $index + 1
      },
      showCreate() {
        //显示新增对话框
        this.tempworkload.content = "";
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
      showUpdate($index) {
        //显示修改对话框
        this.tempworkload.id = this.list[$index].id;
        this.tempworkload.type = this.list[$index].type;
        this.tempworkload.hours = this.list[$index].hours;
        this.tempworkload.content = this.list[$index].content;
        this.dialogStatus = "update"
        this.dialogFormVisible = true
      },
      showDelete($index) {
        //显示修改对话框
        this.tempworkload.id = this.list[$index].id;
        this.tempworkload.type = this.list[$index].type;
        this.tempworkload.hours = this.list[$index].hours;
        this.tempworkload.content = this.list[$index].content;
        this.dialogStatus = "delete"
        this.dialogFormVisible = true
      },
      createworkload() {
        //保存新
        this.api({
          url: "/workload/addworkload",
          method: "post",
          data: this.tempworkload
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      updateworkload() {
        //修改
        this.api({
          url: "/workload/updateworkload",
          method: "post",
          data: this.tempworkload
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      deleteworkload() {
        //删除
        this.api({
          url: "/workload/deleteworkload",
          method: "post",
          data: this.tempworkload
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
    }
  }
</script>
