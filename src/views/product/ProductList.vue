<template>
  <el-row style="display: flex;align-items: center;">
    <el-icon :size="20" ><Edit/></el-icon><span>商品管理</span>
  </el-row>
  <el-row>
    <span style="height: 1px;width: 100%;margin:20px 0;background-color: #ffffff"></span>
  </el-row>
  <!--  <el-row>-->
  <el-form :model="form" ref="form" :inline="true">
    <el-form-item label="商品名称：" prop="name">
      <el-input
          :maxlength="20"
          v-model="form.name"
          placeholder="请输入商品名称"
      />
    </el-form-item>
    <el-form-item label="状态：" prop="isDeleted">
      <el-select v-model="form.isDeleted" class="m-2" placeholder="请选择状态">
        <el-option value="Y"><el-tag :type="'danger'">已删除</el-tag><span>&nbsp;Y</span></el-option>
        <el-option value="N"><el-tag :type="'success'">未删除</el-tag><span>&nbsp;N</span></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="创建时间：" prop="start">
      <el-date-picker
          v-model="form.start"
          value-format="YYYY-MM-DD"
          type="datetime"
          placeholder="请选择开始时间"
      />
    </el-form-item>
    <el-form-item>
      <span>-</span>
    </el-form-item>
    <el-form-item prop="end">
      <el-date-picker
          v-model="form.end"
          value-format="YYYY-MM-DD"
          type="datetime"
          placeholder="请选择结束时间"
      />
    </el-form-item>
    <el-form-item>
      <el-button type="primary" plain @click="search" ><el-icon><Search/></el-icon><span>查询</span></el-button>
      <el-button type="primary" plain @click="formReset" ><el-icon><Refresh /></el-icon><span>重置</span></el-button>
      <el-button type="primary" plain @click="addDialog = true" ><el-icon><Plus /></el-icon><span>新增</span></el-button>
    </el-form-item>
  </el-form>
  <el-table :data="tableData" :row-class-name="rowClass" :table-layout="'auto'" height="1000px">
    <!--    <el-table-column prop="" label="" type="selection"></el-table-column>-->
    <el-table-column prop="id" label="用户ID"></el-table-column>
    <el-table-column prop="name" label="商品名称"></el-table-column>
    <el-table-column prop="barCode" label="商品条码"></el-table-column>
    <el-table-column prop="spec" label="商品规格"></el-table-column>
    <el-table-column prop="unit" label="商品单位"></el-table-column>
    <el-table-column prop="colSpan" label="商品占位"></el-table-column>
    <el-table-column prop="minImageUrl" label="商品小图"></el-table-column>
    <el-table-column prop="isDeleted" label="状态">
      <template #header>
        <el-tooltip
            content="Y：已删除,N：未删除"
            placement="top-start"
            trigger="hover"
        >
          <span>状态</span>
        </el-tooltip>
      </template>
      <template #default="scope">
        <el-tag :type="scope.row.isDeleted === 'N' ? 'success' : 'error'">{{ scope.row.isDeleted }}</el-tag>
      </template>
    </el-table-column>
    <el-table-column prop="createTimeStr" label="创建时间"></el-table-column>
    <el-table-column prop="updateTimeStr" label="更新时间"></el-table-column>
    <el-table-column label="操作">
      <template #default="scope">
        <el-button size="default" plain type="info" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
        <el-button size="default" plain type="danger" @click="handleDelete(scope.$index, scope.row)" v-if="scope.row.isDeleted==='N'" >删除</el-button>
        <el-button size="default" plain type="success" @click="handleOpen(scope.$index, scope.row)" v-else >启用</el-button>
      </template>
    </el-table-column>
  </el-table>

  <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :page-sizes="pageSizes"
      v-model:currentPage="currentPage"
      v-model:page-size="pageSize"
      @prev-click="prePage"
      @next-click="nextPage"
      layout="total, sizes, prev, pager,  next, jumper"
      :total="total"
      v-bind:dataSize="tableData.length"
      :style="paginationStyle"/>
  <!--  新增 -->
  <el-dialog v-model="addDialog" title="新增信息" width="50%" center :destroy-on-close="true" @close="closeDiaLog">
        <el-form :model="addForm" ref="addForm" :rules="rules" @validate="validate(addForm)" label-width="150px" :inline="true">
          <el-form-item label="商品名称" prop="name">
            <el-input type="text" v-model="addForm.name" placeholder="请输入商品名称" />
          </el-form-item>
          <el-form-item label="商品条码" prop="barCode">
            <el-input type="text" v-model="addForm.barCode" placeholder="请输入商品类型"/>
          </el-form-item>
          <el-form-item label="商品类型" prop="typeId">
<!--            <el-input type="text" v-model="addForm.typeId" placeholder="请输入商品类型"/>-->
            <el-select v-model="addForm.typeId"  placeholder="请选择商品类型">
              <el-option
                  v-for="item in productTypeList"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="商品规格" prop="spec">
            <el-input type="text" v-model="addForm.spec" placeholder="请输入商品规格"/>
          </el-form-item>
          <el-form-item label="商品单位" prop="unit">
            <el-input type="text" v-model="addForm.unit" placeholder="请输入商品单位"/>
          </el-form-item>
          <el-form-item label="商品描述" prop="description">
            <el-input type="textarea" v-model="addForm.description" placeholder="请输入商品类型"/>
          </el-form-item>
          <el-form-item label="商品占位" prop="colSpan">
            <el-input type="number" v-model="addForm.colSpan" placeholder="商品占位" min="1"/>
          </el-form-item>
          <el-form-item label="商品小图" prop="minImageUrl">
            <el-input type="type" v-model="addForm.minImageUrl" placeholder="请上传商品小图"/>
          </el-form-item>
          <el-form-item label="商品原图" prop="normalImageUrl">
            <el-input type="type" v-model="addForm.normalImageUrl" placeholder="请上传商品原图"/>
          </el-form-item>
        </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="addDialog = false">取消</el-button>
        <el-button type="primary" @click="addProduct(this.addForm)">保存</el-button>
      </span>
    </template>
  </el-dialog>

  <el-dialog v-model="editDialog" title="编辑信息" width="30%" center :destroy-on-close="true">
    <el-form ref="editForm" :model="editForm" :rules="rules" @validate="validate(editForm)" status-icon label-width="100px">
      <el-form-item label="商品名称" prop="name">
        <el-input v-model="editForm.name" placeholder="请输入商品名称" />
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="editDialog = false">取消</el-button>
        <el-button type="primary" @click="editProduct(this.editForm)">保存</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script >
import { markRaw } from 'vue';
import { Delete } from '@element-plus/icons-vue';
export default {
  name: "ProductList",
  data(){
    return {
      currentPage:1,
      tableData:[],
      productTypeList:[],
      pageSize:10,
      pageSizes:[10, 20, 30, 40, 50, 100],
      total:0,
      form:{
        name:'',
        isDeleted:'',
        start:'',
        end:'',
      },
      addForm:{
        name:'',
        barCode:'',
        typeId:'',
        spec:'',
        unit:'',
        description:'',
        minImageUrl:'',
        normalImageUrl:'',
        colSpan:1
      },
      editForm:{
        id:'',
        name:''
      },
      paginationStyle:{
        marginTop:'20px'
      },
      addDialog:false,
      editDialog:false,
      rules: {
        name: [{ required: true, message: "请输商品名称", trigger: "blur" }],
        barCode: [{ required: true, message: "请输用商品条码", trigger: "blur" }],
        typeId: [{ required: true, message: "请输用商品类型", trigger: "blur" }],
        spec: [{ required: true, message: "请输用商品规格", trigger: "blur" }],
        unit: [{ required: true, message: "请输用商品单位", trigger: "blur" }],
        colSpan: [{ required: true, message: "请输用商品占位", trigger: "blur" }],
      }
    }
  },
  created() {
    this.getProductList();
    this.getProductTypeList();
  },
  methods:{
    getProductList(){
      const page = {
        page:this.currentPage,
        pageSize:10
      }
      this.axios.post(this.HttpRequestApi.terminal_productManage,this.$Qs.stringify(page))
          .then(resp=>{
            if (resp.data.success){
              this.tableData = resp.data.data.list
              this.total = resp.data.data.total
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    getProductTypeList(){
      this.axios.post(this.HttpRequestApi.terminal_productManage_productTypeList)
          .then(resp=>{
            if (resp.data.success){
              this.productTypeList = resp.data.data
              console.log(this.productTypeList)
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    rowClass(row) {
      if (row.rowIndex%2 === 0) {
        return 'warning-row'
      } else {
        return 'success-row'
      }
    },
    nextPage(page){
      const pager = {
        page:page+1,
        pageSize:10
      }
      this.axios.post(this.HttpRequestApi.terminal_productManage,this.$Qs.stringify(pager))
          .then(resp=>{
            if (resp.data.success){
              this.tableData = resp.data.data.list
              this.total = resp.data.data.total
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    prePage(page){
      const pager = {
        page:page-1,
        pageSize:10
      }
      this.axios.post(this.HttpRequestApi.terminal_productManage,this.$Qs.stringify(pager))
          .then(resp=>{
            if (resp.data.success){
              this.tableData = resp.data.data.list
              this.total = resp.data.data.total
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    handleSizeChange(newSize){
      const pager = {
        page:this.currentPage,
        pageSize:newSize
      }
      this.axios.post(this.HttpRequestApi.terminal_productManage,this.$Qs.stringify(pager))
          .then(resp=>{
            if (resp.data.success){
              this.tableData = resp.data.data.list
              this.total = resp.data.data.total
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    handleCurrentChange(newPage){
      const pager = {
        page:newPage,
        pageSize:this.pageSize
      }
      this.axios.post(this.HttpRequestApi.terminal_productManage,this.$Qs.stringify(pager))
          .then(resp=>{
            if (resp.data.success){
              this.tableData = resp.data.data.list
              this.total = resp.data.data.total
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    search(){
      const pager = {
        page:1,
        pageSize:this.pageSize,
        name:this.form.name,
        isDeleted:this.form.isDeleted,
        start:this.form.start,
        end:this.form.end,
      }
      this.axios.post(this.HttpRequestApi.terminal_productManage,this.$Qs.stringify(pager))
          .then(resp=>{
            if (resp.data.success){
              this.tableData = resp.data.data.list
              this.total = resp.data.data.total
            }
          }).catch(error=>{
        console.log(error)
      })
    },
    handleEdit(index,row){
      this.editForm.name = row.name;
      this.editForm.id = row.id;
      this.editDialog = true;
    },
    handleDelete(index,row){
      this.$messageBox.confirm(
          '删除用户：'+row.name,
          '确认删除吗?',
          {
            type:'warning',
            icon:markRaw(Delete),
            center:true,
            cancelButtonText:'取消',
            confirmButtonText:'确认',
            draggable:true,
          }).then(()=>{
        this.axios.post(this.HttpRequestApi.terminal_productManage_del,this.$Qs.stringify({id:row.id}))
            .then(resp=>{
              if (resp.data.success){
                this.$message({
                  message: '删除成功',
                  type: 'success',
                  center:true,
                  duration:1500
                })
                this.search();
              } else {
                this.$message({
                  message: '删除失败，请重试',
                  type: 'error',
                  center:true,
                  duration:1500
                })
              }
            }).catch(error=>{
          this.$message({
            message: '删除失败，请重试',
            type: 'error',
            center:true,
            duration:1500
          })
          console.log(error)
        })
      }).catch(()=>{
        //todo
      })
      console.log(index,row)
    },
    validate:function (form) {
      if (form.name==''){
        return false;
      }
      if (form.type==''){
        return false;
      }
      if (form.checkPass==''){
        return false;
      }
      return true;
    },
    addProduct(form){
      this.$refs.addForm.validate((valid)=> {
        if (valid){
          const adminProduct = {
            name:form.name,
            type:form.type,
            barCode:form.barCode,
            typeId:form.typeId,
            spec:form.spec,
            unit:form.unit,
            description:form.description,
            minImageUrl:form.minImageUrl,
            normalImageUrl:form.normalImageUrl,
            colSpan:form.colSpan
          }
          this.axios.post(this.HttpRequestApi.terminal_productManage_add,this.$Qs.stringify(adminProduct))
              .then(resp=>{
                if (resp.data.success){
                  this.$message({
                    message: '保存成功',
                    type: 'success',
                    center:true,
                    duration:1500
                  })
                  this.addDialog = false
                  this.search();
                } else {
                  this.$message({
                    message: resp.data.msg,
                    type: 'error',
                    center:true,
                    duration:1500
                  })
                }
              }).catch(error=>{
            this.$message({
              message: '保存失败，请重试',
              type: 'error',
              center:true,
              duration:1500
            })
            console.log(error)
          })
        }
      })
    },
    editProduct(form){
      this.$refs.editForm.validate((valid)=> {
        if (valid){
          const adminProduct = {
            id:form.id,
            name:form.name
          }
          this.axios.post(this.HttpRequestApi.terminal_productManage_update,this.$Qs.stringify(adminProduct))
              .then(resp=>{
                if (resp.data.success){
                  this.$message({
                    message: '保存成功',
                    type: 'success',
                    center:true,
                    duration:1500
                  })
                  this.editDialog = false
                  this.search();
                } else {
                  this.$message({
                    message: '保存失败，请重试',
                    type: 'error',
                    center:true,
                    duration:1500
                  })
                }
              }).catch(error=>{
            this.$message({
              message: '保存失败，请重试',
              type: 'error',
              center:true,
              duration:1500
            })
            console.log(error)
          })
        }
      })
    },
    formReset(){
      this.$refs.form.resetFields()
    },
    handleOpen(index,row){
      this.$messageBox.confirm(
          '启用用户：'+row.name,
          '确认启用吗?',{
            type:'warning',
            icon:markRaw(Delete),
            center:true,
            cancelButtonText:'取消',
            confirmButtonText:'确认',
            draggable:true,
          }).then(()=>{
        const product = {
          id:row.id,
          isDeleted:'N'
        }
        this.axios.post(this.HttpRequestApi.terminal_productManage_update,this.$Qs.stringify(product))
            .then(resp=>{
              if (resp.data.success){
                this.$message({
                  message: '启用成功',
                  type: 'success',
                  center:true,
                  duration:1500
                })
                this.search();
              } else {
                this.$message({
                  message: '启用失败，请重试',
                  type: 'error',
                  center:true,
                  duration:1500
                })
              }
            }).catch(error=>{
          this.$message({
            message: '启用失败，请重试',
            type: 'error',
            center:true,
            duration:1500
          })
          console.log(error)
        })
      }).catch(()=>{
        //todo
      })
    },
    closeDiaLog(){
      this.$refs.addForm.resetFields()
    }
  },
}



</script>

<style>
.el-popper.is-dark{
  background: var(--el-text-color-regular);
}
</style>