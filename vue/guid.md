# 基本语法
```
创建new Vue({})
绑定容器：el: '#app',
变量：data: {
    message: 'Hello Vue!'
  }
绑定属性：v-bind:属性名 / :属性名（单项绑定）
双向绑定值：v-model=""
方法：要写到methods:{}中，
语法：方法名(){} / 方法:function(){}
if条件：v-if=""
for循环：v-for="临时名 in 数组" 或 v-for="(临时名,次数名) in 数组"
click事件：@click="" 或 v-on:click=""
```
# 框架搭建
> 命令
```cmd
#注册cnpm 速度快
npm install -g cnpm --registry=https://registry.npm.taobao.org
#安装vue脚手架
cnpm install vue-cli –g
#初始化vue项目
vue init webpack 项目名称
```

# 项目结构
1. 页面  
    单页面模式，只有一个页面叫index.html，里面的内容我们无须做调整。  
2. 程序执行入口  
    main.js  
    在入口中，注入了路由管理和组件管理，它负责调配在页面中渲染什么内容。  
3. 路由管理  
    router/index.js  
    描述路由和组件之间的对应关系  
4. 中介组件  
    app.vue  
    他被挂载在main.js中，他是默认加载到页面上去的。  
    当路由变更时，对应的组件加载到App.vue中的 \<router-view/>内    
5. 资源文件存放位置  
    src/assets目录下，这里适合静态资源  
    static目录下，适合动态资源（通过变量引用，需要在组件中切换的） 

# 组件及其工具
1. 语法提示插件  
vetur
2. 格式化代码插件  
beautify
3. 快速构建vue页面  
   快捷键 Ctrl+shift+p  
   输入 snipet 选择首选项并点击  
   然后再输入vue  
   打开vue.json粘贴如下内容:  

   ```json
   {
    "Print to console": {
        "prefix": "vue",
        "body": [
            "<!-- $1 -->",
            "<template>",
            "<div class='$2'>$5</div>",
            "</template>",
            "",
            "<script>",
            "//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）",
            "//例如：import 《组件名称》 from '《组件路径》';",
            "",
            "export default {",
            "//import引入的组件需要注入到对象中才能使用",
            "components: {},",
            "data() {",
            "//这里存放数据",
            "return {",
            "",
            "};",
            "},",
            "//监听属性 类似于data概念",
            "computed: {},",
            "//监控data中的数据变化",
            "watch: {},",
            "//方法集合",
            "methods: {",
            "",
            "},",
            "//生命周期 - 创建完成（可以访问当前this实例）",
            "created() {",
            "",
            "},",
            "//生命周期 - 挂载完成（可以访问DOM元素）",
            "mounted() {",
            "",
            "},",
            "beforeCreate() {}, //生命周期 - 创建之前",
            "beforeMount() {}, //生命周期 - 挂载之前",
            "beforeUpdate() {}, //生命周期 - 更新之前",
            "updated() {}, //生命周期 - 更新之后",
            "beforeDestroy() {}, //生命周期 - 销毁之前",
            "destroyed() {}, //生命周期 - 销毁完成",
            "activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发",
            "}",
            "</script>",
            "<style lang='scss' scoped>",
            "//@import url($3); 引入公共css类",
            "$4",
            "</style>"
        ],
        "description": "Log output to console"
    }
}
```

 建立vue组件，输入vue按TAB即可快速生成文件结构

# vue对象  
常用属性说明  
```javascript
export default {
    //import引入的组件需要注入到对象中才能使用
    components: {},
    data() {
      //这里存放数据
      return {
        user: {},
        categorys: [],
        childrenCate: [],
        pageBean:{},
        queryParams:{
          page:1,
          rows:2
        }
      };
    },  
    computed: {},//监听属性 类似于data概念
    watch: {},//监控data中的数据变化
    methods: { },//方法集合
    created() { }, //生命周期 - 创建完成（可以访问当前this实例）
    mounted() {},//生命周期 - 挂载完成（可以访问DOM元素）
    beforeCreate() {}, //生命周期 - 创建之前
    beforeMount() {}, //生命周期 - 挂载之前
    beforeUpdate() {}, //生命周期 - 更新之前
    updated() {}, //生命周期 - 更新之后
    beforeDestroy() {}, //生命周期 - 销毁之前
    destroyed() {}, //生命周期 - 销毁完成
    activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
  }
```
# 数据渲染
1. 标签内容渲染
```vue
<template>
    <div>
        欢迎你：{{name}}
    </div>
</template>
<script>
    export default{
        data(){
            return {
                name:'张三'
            }
        }
    }
</script>
```
2. 标签属性渲染
```vue
<template>
    <div>
        <img :src='imgsrc'/>
    </div>
</template>
<script>
    export default{
        data(){
            return {
                src:'static/image1.png'
            }
        }
    }
</script>
```
属性值可以为表达式
```vue
<template>
    <div :class="page>1?'on':'off'">
        <img :src='imgsrc'/>
    </div>
</template>
<script>
    export default{
        data(){
            return {
                src:'static/image1.png',
                page:2
            }
        }
    }
</script>
<style>
    .on{
        background:red
    }
    .off{
        background:blue
    }
</style>
```
3. 双向绑定
```vue
<template>
    <div>
        姓名：<input type='text' v-model='name'>
    </div>
</template>
<script>
    export default{
        data(){
            return {
                name:'张三'
            }
        }
    }
</script>
```
> 数组绑定不刷新问题  
支持刷新的方法：push pop  shift unshift splice