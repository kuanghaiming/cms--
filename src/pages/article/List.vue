<template>
  <div class="article_list">
    <!-- 按钮 -->
    <div class="btns">
      <el-button type="primary" size="small" @click="toPublishArticle">添加文章</el-button>
      <el-button type="danger" size="small" @click="toBatchDelete">批量删除</el-button>
    </div>
    <!-- 表格 data动态变量  @selection-change="handleSelectionChange" 回调记录批量删除的编号 在批量删除中用到-->
    {{ids}}
    <el-table :data="articles" style="width: 100%" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="编号" width="100">
      </el-table-column>
      <el-table-column prop="title" label="标题" width="180">
      </el-table-column>
      <el-table-column prop="authorId" label="作者">
      </el-table-column>
      <el-table-column prop="category.name" label="所属栏目">
      </el-table-column>
      <el-table-column prop="category." label="父栏目">
      </el-table-column>

      <el-table-column
        fixed="right"
        label="操作"
        align="center"
        width="200">
        
        <template slot-scope="scope">
          <el-button icon="el-icon-search" @click="toReview(scope.row)" type="text" size="small">查看</el-button>
          <el-button icon="el-icon-edit" @click="toEdit(scope.row)" type="text" size="small">编辑</el-button>
          <el-button icon="el-icon-delete" @click="toDelete(scope.row.id)" type="text" size="small">删除</el-button>
        </template>
        </el-table-column>
      </el-table>
      
      <!-- 查看模态框 -->
      <el-dialog title="新增栏目信息" :visible.sync="dialogTableVisible">
        {{form.id}}
        <el-table :data="form" style="width: 100%" >
          <el-table-column prop="id" label="编号" width="100">
          </el-table-column>
          <!-- <el-table-column prop="title" label="标题" width="180">
          </el-table-column>
          <el-table-column prop="content" label="内容" width="180">
          </el-table-column>
          <el-table-column prop="source" label="来源" width="180">
          </el-table-column> -->
          <!-- <el-table-column prop="publish_time" label="提交时间" width="180">
          </el-table-column>
          <el-table-column prop="read_times" label="查看时间" width="180">
          </el-table-column>
          <el-table-column prop="status" label="状态" width="180">
          </el-table-column>
          <el-table-column prop="thump_up" label="点赞数" width="180">
          </el-table-column>
          <el-table-column prop="thump_down" label="否认数" width="180">
          </el-table-column>
          <el-table-column prop="authorId" label="作者">
          </el-table-column>
          <el-table-column prop="category.name" label="所属栏目">
          </el-table-column>
          <el-table-column prop="category." label="父栏目">
          </el-table-column> -->

        </el-table>

      </el-dialog>
      <!-- 新增模态框 -->

      <!-- 分页 -->
      <div class="block" align="center">
        
        <el-pagination  
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
      articles:[],
      ids:[],
      currentPage4: 4,
      total:"", 
      packNumber:5,
      offset:0,
      dialogTableVisible:false,
      form:{},
    }
  },
  //生命周期
  created(){
    //调用ajax查询所有文章数据
    this.reloadData();

    let url = "http://127.0.0.1:8090/article/selectCount"
      request
      .get(url)
      .then(result => {
        this.total = result.data;
    })
  },
  //方法 处理事件
  methods:{
    //复选框回调
    handleSelectionChange(val){
      this.ids = val.map(item=>item.id);  
    },

    //重载数据  get不能通过 get(url,{total:this.total}) 因为get传的是字符串
    //  (url,{total:this.total})只能用post传 传的是Json
    reloadData(){
      // 调用ajax查询所有文章数据
      
      let limit =  this.packNumber;
      let url = "http://127.0.0.1:8090/article/findArticleByPagination"
        +"?offset="+this.offset+"&limit="+limit
      request.get(url)
      .then(result=>{
        this.articles = result.data;
      })
    },

    toReview(val){
      // this.$router.push({path:''});
      this.form=val;
      this.dialogTableVisible=true;
    },

    //record 修改那一行
    toEdit(record){
        console.log(record)
        //query 查询字符串 query:record 可以把这一行的信息传入给editor
        this.$router.push({path:'/article/editor',query:record})
    },
    toPublishArticle(){
      // 跳转到编辑页面  并把
      this.$router.push({path:'/article/editor'})
    },

    toDelete(id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let url = 'http://127.0.0.1:8090/article/deleteById'
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
      let url = "http://127.0.0.1:8090/article/batchDelete"
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
    toFindArticleByPage(val){
      if(val==1){
        this.offset = 0;
        this.PageQuery(0,this.packNumber);
      }else{
        this.offset = (val-1)*this.packNumber;
        this.PageQuery(this.offset,this.packNumber);
      }
      
    },

    PageQuery(offset,limit){
      let url = "http://127.0.0.1:8090/article/findArticleByPagination"
        +"?offset="+offset+"&limit="+limit
      request.get(url)
      .then(result=>{
        this.articles = result.data;
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
      this.toFindArticleByPage(val);
    },

    //修改或删除
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

  }
}
</script>
<style scoped>

</style>