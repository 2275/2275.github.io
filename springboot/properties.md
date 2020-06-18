# properties与 yml
> properties 为扁平结构  
> yml 为层次结构  冒号后面必须有空格，否则报错

``` 示例
#properties配置方法
com.user.dept=开发部
com.user.sex=男

#yml配置方法
com:
    user:
        sex: 男
        dept: 开发部
```
请按照喜好选择！:laughing:
# 自定义变量

``` properties
com.user.dept=开发部
com.user.sex=男
com.user.phone=13812345643,13828317388   //数组类型
com.env.javahomt=${JAVA_HOME}  //可直接调用环境变量
```
>  使用时，在属性上加注解@Value("${com.demo.xx}")即可注入值
```java
@RestController
public class UserController {

    @Value("${com.user.dept}")
    private  String name;
 

    @RequestMapping("/")
    public String show(){
        return "我的部门是："+name
    }
}
```
>  也可以直接绑定对象，但需要进行如下两步设置

```java
@ConfigurationProperties(prefix = "com.user")
public class User {
    private String name;
    private String sex;
    // 省略getter和setter
}

@SpringBootApplication
@EnableConfigurationProperties({User.class})
public class GenApplication {

    public static void main(String[] args) {
        SpringApplication.run(GenApplication.class, args);
    }
}
```
>  定义的变量可以直接在属性文件中引用
``` properties
com.user.dept=开发部
com.user.sex=男
myname=${com.user.dept}
```
> 定义的变量也可以通过代码获取
```java
    /**
     * 通过配置文件名读取内容
     * @param fileName
     * @return
     */
    private static Properties readPropertiesFile(String fileName) {
        try {
            Resource resource = new ClassPathResource(fileName);
            Properties props = PropertiesLoaderUtils.loadProperties(resource);
            return props;
        } catch (Exception e) {
            System.out.println("————读取配置文件：" + fileName + "出现异常，读取失败————");
            e.printStackTrace();
        }
        return null;
    }
 
    public static void main(String[] args) {
        Properties properties = readPropertiesFile("application.properties");
        System.out.println(properties.getProperty("name"));
    }
```
# 多环境配置
> 命名规范：  
> application-dev.properties  //开发环境  
> application-prod.properties   //生产环境  
> 在application.properties中设置 spring.profiles.active属性

``` properties
spring.profiles.active=dev  // 调用application-dev.properties
``` 
> 该方法也可以用来做属性文件拆分  
> application-mvc.properties  //MVC配置  
> application-dao.properties   //DAO配置  
> application-log.properties   //日志配置  
``` properties
spring.profiles.active=mvc,dao,log  // 全部引入，如果有相同变量，则后面的会覆盖前面的
``` 
# 外部配置
> 如果不希望配置打包到jar包中，则需要把配置文件放在类路径外，加载方式也会改变
```properties
spring.config.additional-location="C:/config/"
```
