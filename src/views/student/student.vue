<template>
  <div student="app-container">
    <div student="filter-container">
      <el-form>
        <el-form-item>
          <el-button type="primary" icon="plus" @click="showCreate" v-if="hasPerm('student:add')">添加
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
      <el-table-column align="center" prop="name" label="姓名" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="phone" label="手机号" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="type" label="性别" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="grade" label="年级" style="width: 60px;"></el-table-column>
      <el-table-column align="center" prop="content" label="内容" style="width: 120px;"></el-table-column>
      <el-table-column align="center" prop="createBy" label="创建人" style="width: 60px;"></el-table-column>
      <el-table-column align="center" label="创建时间" width="170">
        <template slot-scope="scope">
          <span>{{scope.row.createTime}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="管理" width="200" v-if="hasPerm('student:update') || hasPerm('student:delete')">
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
      <el-form student="small-space" :model="tempstudent" label-position="left" label-width="60px"
               style='width: 300px; margin-left:50px;'>

        <el-form-item label="姓名">
          <el-input type="text" v-model="tempstudent.name">
          </el-input>
        </el-form-item>

        <el-form-item label="手机号">
          <el-input type="text" v-model="tempstudent.phone">
          </el-input>
        </el-form-item>

        <el-form-item label="性别">
          <el-select v-model="tempstudent.type" placeholder="请选择">
            <el-option
              value="男">
            </el-option>
            <el-option
              value="女">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="年级">
          <el-select v-model="tempstudent.grade" placeholder="请选择">
            <el-option
              value="大一">
            </el-option>
            <el-option
              value="大二">
            </el-option>
            <el-option
              value="大三">
            </el-option>
            <el-option
              value="大四">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="内容">
          <el-input type="text" v-model="tempstudent.content">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" student="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-if="dialogStatus=='create'" type="success" @click="createstudent">创 建</el-button>
        <el-button v-if="dialogStatus=='update'" type="primary"  @click="updatestudent">修 改</el-button>
        <el-button v-if="dialogStatus=='delete'" v-else type="primary" v-else @click="deletestudent">删 除</el-button>
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
        tempstudent: {
          id: "",
          type:"",
          grade:"",
          subject:"",
          createBy:"",
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
        if (!this.hasPerm('student:list')) {
          return
        }
        this.listLoading = true;
        this.api({
          url: "/student/liststudent",
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
        this.tempstudent.content = "";
        this.dialogStatus = "create"
        this.dialogFormVisible = true
      },
      showUpdate($index) {
        //显示修改对话框
        this.tempstudent.id = this.list[$index].id;
        this.tempstudent.type = this.list[$index].type;
        this.tempstudent.grade = this.list[$index].grade;
        this.tempstudent.phone = this.list[$index].phone;
        this.tempstudent.name = this.list[$index].name;
        this.tempstudent.content = this.list[$index].content;
        this.dialogStatus = "update"
        this.dialogFormVisible = true
      },
      showDelete($index) {
        //显示修改对话框
        this.tempstudent.id = this.list[$index].id;
        this.tempstudent.type = this.list[$index].type;
        this.tempstudent.grade = this.list[$index].grade;
        this.tempstudent.phone = this.list[$index].phone;
        this.tempstudent.name = this.list[$index].name;
        this.tempstudent.content = this.list[$index].content;
        this.dialogStatus = "delete"
        this.dialogFormVisible = true
      },
      createstudent() {
        //保存新
        this.api({
          url: "/student/addstudent",
          method: "post",
          data: this.tempstudent
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      updatestudent() {
        //修改
        this.api({
          url: "/student/updatestudent",
          method: "post",
          data: this.tempstudent
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
      deletestudent() {
        //删除
        this.api({
          url: "/student/deletestudent",
          method: "post",
          data: this.tempstudent
        }).then(() => {
          this.getList();
          this.dialogFormVisible = false
        })
      },
    }
  }
</script>
