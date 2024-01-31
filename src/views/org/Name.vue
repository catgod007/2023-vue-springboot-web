<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0;">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.name" placeholder="店铺名" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="search">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleAdd" >新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <!--@selection-change
        勾选才会触发，会把自己勾选的数据选中-->
    <!--列表-->
    <el-table :data="shops" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" width="60">
      </el-table-column>
      <el-table-column prop="name" label="店铺名称" width="120" sortable>
      </el-table-column>
      <el-table-column prop="state" label="店铺状态" width="120" sortable>
        <template slot-scope="scope">
          <span style="color: #13ce66" v-if="scope.row.state == 0">正常</span>
          <span style="color:red" v-else>禁用</span>
        </template>
      </el-table-column>

      <el-table-column prop="tel" label="店铺电话" width="120" sortable>
      </el-table-column>

      <el-table-column prop="address" label="店铺地址" width="250" sortable>
      </el-table-column>

      <el-table-column prop="logo" label="Logo" width="120" sortable>
      </el-table-column>

      <el-table-column prop="registerTime" label="创建时间" width="120" sortable>
      </el-table-column>

      <el-table-column prop="admin.username" label="管理员" width="120" sortable>
      </el-table-column>

      <el-table-column label="操作" width="150">
        <template scope="scope">
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
      <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange"
                     :page-size="pageSize"
                     :total="total"
                     :current-page="currentPage"
                     style="float:right;">
      </el-pagination>
    </el-col>

    <!--编辑界面-->
    <el-dialog :title=title :visible.sync="editFormVisible" :close-on-click-modal="false">
      <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
        <el-form-item label="店铺名称" prop="name">
          <el-input v-model="editForm.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="店铺状态">
          <el-radio-group v-model="editForm.state" auto-complete="off">
            <el-radio class="radio" :label="-1">禁用</el-radio>
            <el-radio class="radio" :label="0">正常</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="联系方式">
          <el-input v-model="editForm.tel" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="创建时间" >
          <el-date-picker v-model="editForm.regisTertime"  type="datetime" placeholder="选择日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="店铺地址">
          <el-input v-model="editForm.address" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="Logo">
          <el-input  v-model="editForm.logo" auto-complete="off"></el-input>
<!--          <el-upload v-model="editForm.logo"-->
<!--                     class="upload-demo"-->
<!--                     drag-->
<!--                     action="http://localhost:8080/food/upload/"-->
<!--                     multiple>-->
<!--            <i class="el-icon-upload"></i>-->
<!--            <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>-->
<!--            <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500MB</div>-->
<!--          </el-upload>-->
        </el-form-item>


        <el-form-item label="管理员" prop="admin">
          <el-select v-model="editForm.admin" value-key="id" placeholder="请选择">
            <el-option
                v-for="item in employees"
                :key="item.id"
                :label="item.username"
                :value="item">
              <span style="float: left">{{ item.username }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ item.phone }}</span>
            </el-option>
          </el-select>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
      </div>
    </el-dialog>

  </section>
</template>

<script>
export default {
  data() {
    return {
      filters: {
        name: ''
      },
      deptTree: [],
      shops: [],
      employees: [],
      total: 0,
      title: '',
      //page: 1,
      currentPage: 1,
      pageSize: 5,
      listLoading: false,
      sels: [],//列表选中列

      editFormVisible: false,//编辑界面是否显示
      editLoading: false,
      editFormRules: {  //表单校验
        name: [
          {required: true, message: '请输入部门名称', trigger: 'blur'}
        ]
      },
//编辑界面数据
      editForm: {
        id: null,
        name: '',
        address: '',
        tel: '',
        state: 0,
        logo: '',
        registerTime: null,
        admin: null,
      },

      addFormVisible: true,//新增界面是否显示
      addLoading: true,
      addFormRules: {
        name: [
          {required: true, message: '请输入姓名', trigger: 'blur'}
        ]
      },

    }
  },
  methods: {
//性别显示转换
    formatSex: function (row, column) {
      return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
    },
    handleCurrentChange(val) {
      this.currentPage = val;
      this.getShops();
    },
//查询
    search() {
      this.currentPage = 1
      this.getShops();
    },
    //获取商铺列表
    getShops() {
      let para = {
        currentPage: this.currentPage,
        pageSize: this.pageSize,
        keyword: this.filters.name
      };
      this.listLoading = true;//盲等框
      this.$http.post("/name", para).then(result => {
        console.log(result)
        //this.currentPage = 1//查询时从头开始查,用上面的search方法代替
        //将查询的分页数据复制deparments
        this.shops = result.data.rows
       //将总条数复制给total
        this.total = result.data.total
        this.listLoading = false;//盲等框

      })
    },
//删除
    handleDel: function (index, row) {
      this.$confirm('确认删除该记录吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        //NProgress.start();
        this.$http.delete("/name/" + row.id).then(result => {
          console.log(result)
          if (result.data.success) {
            this.$message({
              message: '删除成功',
              type: 'success'
            });
          } else {
            this.$message({
              message: '删除失败!!!',
              type: 'error'
            });
          }
          this.listLoading = false;
          this.search();

        })

      });
    },
//显示编辑界面
    handleEdit: function (index, row) {
      this.title = "编辑";
      this.editFormVisible = true;
      this.editForm = Object.assign({}, row);//克隆当前行的数据显示到模态框（回显数据）
    },
//显示新增界面
    handleAdd: function () {
      this.title = "新增";
      this.editFormVisible = true;
      this.editForm = {//清空数据
        id: null,
        name: '',
        address: '',
        tel: '',
        state: 0,
        logo: '',
        registerTime: null,
        admin: null,
      };
    },
//编辑
    editSubmit: function () {
      let validate = this.$refs.editForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.editLoading = true;
            //NProgress.start();
            let para = Object.assign({}, this.editForm);

            // if (para.parent) {
            //   para.parent = {"id": para.parent[para.parent.length - 1]}//把parent的id处理成对象传给后端 paea.parent={id:2}
            // }

            this.$http.put("/name", para).then(result => {
              console.log(result)
              this.editLoading = false;
              this.editFormVisible = false;
              if (result.data.success) {
                this.$message({
                  message: '操作成功',
                  type: 'success'
                });
              } else {
                this.$message({
                  message: '操作失败!!!',
                  type: 'error'
                });
              }
            })
            this.search();
          });
        }
      })
    },
    selsChange: function (sels) {
      this.sels = sels;
    },
//批量删除
    batchRemove: function () {
      let ids = this.sels.map(item => item.id);//把sels里面对象的所有id筛选出来用一个对象接收
      this.$confirm('确认删除选中记录吗？', '提示', {
        type: 'warning'
      }).then(() => {
        console.log(ids)
        this.listLoading = true;
//NProgress.start();
        this.$http.patch("/name", ids).then(result => { //delete对象不能传对象
          console.log(result)
          if (result.data.success) {
            this.$message({
              message: '删除成功',
              type: 'success'
            });
          } else {
            this.$message({
              message: '删除失败!!!',
              type: 'error'
            });
          }
          this.listLoading = false;
          this.search();
        })
      });
    },
//管理员列表
    getEmployees() {
      this.$http.get("/emp").then(result => {
        console.log(result)
        this.employees = result.data;
      })
    },
    //上级部门及其子类
    getDeptTree() {
      this.$http.get("/dept/tree").then(result => {
        console.log(result)
        this.deptTree = result.data;
      })
    }
  },
  mounted() {
    // this.getDepartments();
    this.getShops();
    this.getEmployees();
    // this.getDeptTree();
//页面加载完毕后，会发送Aixos请求去请求后端的数据。
//axios是vue封装ajax的插件， 新建项目需npm install axios --save 去下载
//axios请求语法：
//axios.请求方式(url地址,你要传递的参数).then(变量名=>{
//    方法体
// })
//实例：
//axios.post("/dept",para).then(result=>{
//  console.log(result)
// })
// }
  },
}


</script>

<style scoped>

</style>