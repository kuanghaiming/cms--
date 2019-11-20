<template>
  <div class="article_editor">
    <el-button type="text" @click="back">返回</el-button>

    <!-- :后面的是数据 @后面的方法 -->
    <!-- {{form}} -->
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="所属栏目">
        <el-select v-model="form.categoryId" placeholder="请选择所属栏目">
          <el-option label="IT" value="3"></el-option>
          <el-option label="娱乐" value="4"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="文章标题">
        <el-input v-model="form.title"></el-input>
      </el-form-item>
      <el-form-item label="正文">
        <el-input type="textarea" v-model="form.content" rows="10"></el-input>
      </el-form-item>
      
    <el-form-item>
        <el-button type="primary" @click="onSubmit">发布</el-button>
      </el-form-item>
    </el-form>

  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
import data from '../../views/pdf/content';
export default {
  data(){
    return {
      form:{}
    }
  },
  // 生命周期创建时 相当于初始化 接受了传来的参数
  created(){
    // 弹出框 验证传来的信息是否有值
    // alert(JSON.stringify(this.$route.query));
    this.form = this.$route.query;
  },

  methods:{
    //返回
    back(){
      this.$router.go(-1);
    },
    onSubmit(){
      //alert(JSON.stringify(this.form))
      //alert(qs.stringify(this.form))
      //通过ajax 将前端收集的数据this.form发送给服务接口
      let url = "http://127.0.0.1:8090/article/saveOrUpdate";
      request.request({
        url,
        method:"post",
        headers:{
          // 告诉服务器我传入的是查询字符串
          "Content-Type":"application/x-www-form-urlencoded"
        },
        data:qs.stringify(this.form)
      })
      .then(result=>{
          this.$message({
          message: '成功发布文章',
          type: 'success'
        });
        this.back();
      })
    }
  }
}
</script>
