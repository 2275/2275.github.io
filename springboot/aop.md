# aop实现记录操作日志
> ### 配置
>>加入aspect依赖
``` pom.xml
<!--加入aspect依赖-->
<dependency>
	<groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.5</version>
</dependency>

<!--转换JSON依赖-->
<dependency>
	<groupId>com.hynnet</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
</dependency>

```

>> 在启动类或配置类上加上注解
``` java
@EnableAspectJAutoProxy
```

>### 创建自动自定义注解
>>添加需要的属性
``` java
@Retention(RetentionPolicy.RUNTIME)//设置注解生命周期
@Target({ElementType.METHOD,ElementType.TYPE})//设置注解可以使用在方法和类上
public @interface Log {				
    String module(); //所属模块
    String description() default ""; //描述
}

```
>### 创建aspect类
>>创建通知方法，记录每次操作信息
``` java
@Aspect //加上aspect注解
@Component //交给spring
public class LogAspect {
	//可以获取返回值	
    @AfterReturning(pointcut = "execution(public * com.bdqn.controller.*.*(..))", returning = "result") //返回值名
    public void AfterReturning(JoinPoint jp, Object result) {
        Log log = jp.getTarget().getClass().getAnnotation(Log.class); //获取执行类上的Log注解
        MethodSignature sm = (MethodSignature) jp.getSignature();
        Log log2 = sm.getMethod().getAnnotation(Log.class); //获取执行方法上的Log注解
	    //获取request 
        HttpServletRequest request = ((ServletRequestAttributes)
         RequestContextHolder.getRequestAttributes()).getRequest();
        System.out.println("操作模块：" + log.module() + "/" + log2.module());//通过注解获取属性
        System.out.println("操作描述：" + log2.description()); 
        System.out.println("请求方式："+request.getMethod());  //request获取请求方式
        System.out.println("请求地址："+request.getRequestURI()); //request获取请求地址 
        //拼接操作方法全名
        System.out.println("操作方法："+jp.getTarget().getClass().getName()+"."+sm.getMethod().getName()+"()"); 
        System.out.println("参数：" + JSONArray.fromObject(jp.getArgs()));
        System.out.println("返回值：" + JSONObject.fromObject(result));
        System.out.println("操作状态：正常");
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("操作时间："+df.format(System.currentTimeMillis()));
    }
	//报错之后执行
    @AfterThrowing(pointcut = "execution(public * com.bdqn.controller.*.*(..))", throwing = "ex")//异常参数
    public void AfterReturning(JoinPoint jp, Exception ex) {
        Log log = jp.getTarget().getClass().getAnnotation(Log.class);
        MethodSignature sm = (MethodSignature) jp.getSignature();
        Log log2 = sm.getMethod().getAnnotation(Log.class);
        HttpServletRequest request = ((ServletRequestAttributes)
         RequestContextHolder.getRequestAttributes()).getRequest();
        System.out.println("操作模块：" + log.module() + "/" + log2.module());
        System.out.println("操作描述：" + log2.description());
        System.out.println("请求方式："+request.getMethod());
        System.out.println("请求地址："+request.getRequestURI());
        System.out.println("操作方法："+jp.getTarget().getClass().getName()+"."+sm.getMethod().getName()+"()");
        System.out.println("参数：" + JSONArray.fromObject(jp.getArgs()));
        System.out.println("操作状态：异常："+ex.getMessage());
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("操作时间："+df.format(System.currentTimeMillis()));
    }

}

```