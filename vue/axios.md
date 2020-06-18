## 基本使用
>安装axios
``` cmd
npm install axios
```
>配置axios

main.js
``` js
import axios from' axios
Vue . prototype.$axios = axios;

```
get
``` js
      this.$axios.get("/user?id=12345")//请求路径?参数
      .then(function(response){//then 请求成功回调函数
        console.log(response);
      })
      .catch(function(error){//cath 请求失败回调函数
        console.log(error);
      });

      this.$axios.get("/user?id=12345",{
        params:{//也可以使用这种方式传参
          id:12345
        }
      })
      .then(function(response){
        console.log(response);
      })
      .catch(function(error){
        console.log(error);
      });
```
post
``` js
 this.$axios.post("/user",{//请求路径
        id:12345//参数
      })
      .then(function(response){
        console.log(response);
      })
      .catch(function(error){
        console.log(error);
      });
```
## 使用拦截器处理返回信息
>创建一个js
``` js
import Vue from 'vue'
import Axios from 'axios'
import qs from 'qs'

import Element from 'element-ui'//引入Element
import 'element-ui/lib/theme-chalk/index.css'//引入Element样式
Vue.use(Element)//使用Element
const tempVue = new Vue();//创建一个空Vue
//拦截响应
Axios.interceptors.response.use(function (response) {
    
    var message = response.data.message;
    if(response.status && response.status == 200 && response.data.code != 500){
        tempVue.$message({message:message,type:"success"})//用空Vue调用Element的message
    }else if(response.status && response.status == 200 && response.data.code == 500){
        tempVue.$message({message:message,type:"error"})
    }
    else{
        tempVue.$message({message:message,type:"error"})
    }
    return response;
  }, function (error) {
    if(error.status == 500){
        tempVue.$message({message:message,type:"error"})
    }else if(error.status == 404){
        tempVue.$message({message:message,type:"error"})
    }
    return error;
  });
//拦截请求
Axios.interceptors.request.use(function (config) {
    return config;
  }, function (error) {
    return Promise.reject(error);
  });
  const axiosPost = (url,params)=>{
        return Axios({
            method:"post",
            url:`${url}`,
            data:qs.stringify(params,{arrayFormat: 'repeat'})
        });
  }
  const axiosGet = (url,params)=>{
    return Axios({
        method:"get",
        url:`${url}`,
        params:params
    });
}
Axios.defaults.baseURL = process.env.ROOT_URL.toString();
Vue.prototype.$axios = Axios;
Vue.prototype.$axiosPost = axiosPost;//将axiosPost属性方法放到Vue属性中
Vue.prototype.$axiosGet = axiosGet;
```
>引入

main.js
``` js
import './util/axios.js'

```
>使用
``` js
        this.$axiosGet("/query",{id:12345}).then(result=>{  
            //回调函数
        });
        this.$axiosPost("/query",{id:12345}.then((result)=>{
            //回调函数
        });
```