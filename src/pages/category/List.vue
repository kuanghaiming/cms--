<template>
  <div class="category_list">
    <!-- 按钮区域 -->
    <div class="btns">
      <el-button type="primary" size="small" @click="toAdd">添加栏目</el-button>
      <el-button type="danger" size="small" @click="toBatchDelete">批量删除</el-button>
    </div>
   
    {{ids}}
    <el-table :data="categorys" style="width: 100%" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="编号" width="100">
      </el-table-column>
      <el-table-column prop="name" label="栏目名称" width="180">
      </el-table-column>
      <el-table-column prop="description" label="栏目描述">
      </el-table-column>
      <el-table-column prop="parent_id" label="父栏目">
      </el-table-column>

      <el-table-column
        fixed="right"
        label="操作"
        align="center"
        width="200">
        <template slot-scope="scope">
          
          <el-button icon="el-icon-edit" @click="toEdit(scope.row)" type="text" size="small">编辑</el-button>
          <el-button icon="el-icon-delete" @click="toDelete(scope.row.id)" type="text" size="small">删除</el-button>
        </template>
        </el-table-column>
    </el-table>
      
      <!-- 新增模态框 -->
      <el-dialog title="新增栏目信息" :visible.sync="dialogTableVisible">
        <el-form :model="form">
          <el-form-item label="父栏目" label-width="80px">
            <el-select v-model="form.parent_id" placeholder="请选择所属栏目">
              
              <el-option v-for="c in this.categorys" 
                        :key="c.id" 
                        :label="c.name" 
                        :value="c.id"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="no编号">
            <el-input  v-model="form.no"></el-input>
          </el-form-item>
          <el-form-item label="栏目名称">
            <el-input v-model="form.name"></el-input>
          </el-form-item>

          <el-form-item label="标题描述">
            <el-input type ="textarea" v-model="form.description" rows="7"></el-input>
          </el-form-item>

        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button size="small" @click="dialogTableVisible = false">取 消</el-button>
          <el-button size="small" type="primary" @click="toSaveOrUpdateSummit()" >确 定</el-button>
        </div>
      </el-dialog>
      <!-- 新增模态框 -->

      <!-- 分页 -->
      <div class="block" align="center">
        
        <el-pagination :data="a" 
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="1"
          :page-sizes="[5, 10, 20, 40]"
          :page-size="100"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>

  </div>
</template>
<script>
// 引入这个可以和后台交互
import request from '@/utils/request'
import qs from 'querystring'
export default {
  // 为模板提供数据
  data(){
    return {
      categorys:[],
      ids:[],
      currentPage4: 4,
      total:"", 
      packNumber:5,
      offset:0,
      form:{},
      dialogTableVisible:false,
    }
  },
  //生命周期
  created(){

    //调用ajax查询所有文章数据
    this.reloadData();

    let url = "http://127.0.0.1:8090/category/selectCount"
      request
      .get(url)
      .then(result => {
        this.total = result.data;
    })
  },
  //方法 处理事件
  methods:{
    

    toSaveOrUpdateSummit(){
        request.request({
          url:'http://127.0.0.1:8090/category/saveOrUpdate',
          method:'post',
          headers:{
            "Content-Type":"application/x-www-form-urlencoded"
          },
          data:qs.stringify(this.form)
        })
        .then(response =>{
          this.$message({
          type: 'success',
          message:'添加成功'
          })
          //关闭模态框 并重载数据
          this.dialogTableVisible = false;
          this.reloadData();
        })
    },

    //复选框回调
    handleSelectionChange(val){
      this.ids = val.map(item=>item.id);  
    },

    //重载数据  get不能通过 get(url,{total:this.total}) 因为get传的是字符串
    //  (url,{total:this.total})只能用post传 传的是Json
    reloadData(){
      // 调用ajax查询所有文章数据
      
      let limit =  this.packNumber;
      let url = "http://127.0.0.1:8090/category/findCategoryByPagination"
        +"?offset="+this.offset+"&limit="+limit
      request.get(url)
      .then(result=>{
        this.categorys = result.data;
      })
    },

    //编辑栏目
    toEdit(record){
        console.log(record)
        this.form = record;
        this.dialogTableVisible=true;

    },
    //添加栏目
    toAdd(){
        this.form={};
        this.dialogTableVisible=true;
    },

    toPublishCategory(){
      // 跳转到编辑页面
      this.$router.push({path:'/category/editor'})
    },

    toDelete(id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let url = 'http://127.0.0.1:8090/category/deleteById'
        //request.get(url,{params:{id}})
        //无法这样传id值回来 post传回来的是json对象 json对象只能用对象去接受
        //post要想接受id的值  还需传字符串类型回来 
        //request.post(url,{id:id})
        request.request({
          url,
          method:"post",
          headers:{
            // 告诉服务器我传入的是查询字符串
            "Content-Type":"application/x-www-form-urlencoded"
          },
          data:qs.stringify({id:id})
        })
        .then(response =>{
          this.$message({
          type: 'success',
          message: '删除成功!'
          })
          this.reloadData();
        })
      })
    },

    toBatchDelete(){
      let url = "http://127.0.0.1:8090/category/batchDelete"
      request.post(url,this.ids)
      .then(response=>{
        this.$message({
          message:response.message,
          type:"success"
        })
        this.reloadData();
      })
    },

    //通过分页查询
    toFindCategoryByPage(val){
      if(val==1){
        this.offset = 0;
        this.PageQuery(0,this.packNumber);
      }else{
        this.offset = (val-1)*this.packNumber;
        this.PageQuery(this.offset,this.packNumber);
      }
      
    },

    PageQuery(offset,limit){
      let url = "http://127.0.0.1:8090/category/findCategoryByPagination"
        +"?offset="+offset+"&limit="+limit
      request.get(url)
      .then(result=>{
        this.categorys = result.data;
      })
    },
    
    
    //分页 时刻更新
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.packNumber = val;
      this.reloadData();
      
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.toFindCategoryByPage(val);
    }

  }
}
</script>
<style scoped>

</style>