# SpringBoot整合Shiro
## 使用
>导入依赖
``` xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.5.3</version>
</dependency>
```
>查询角色和权限
``` sql
@Select("select r.role_name from sys_role r,sys_user u,sys_user_role ur where r.role_id = ur.role_id and u.user_id = ur.user_id and u.user_name = #{username}")
public Set<String> queryRoleByUserName(String username);
@Select("select m.menu_name from sys_role r,sys_user u,sys_menu m,sys_user_role ur,sys_role_menu rm where r.role_id = ur.role_id and u.user_id = ur.user_id and r.role_id = rm.role_id and m.menu_id = rm.menu_id and u.user_name = #{username}")
public Set<String> queryPermitByUserName(String username);
```

>自定义Realm用于查询用户的角色和权限信息并保存到权限管理器：
``` java
@Component
public class MyRealm extends AuthorizingRealm {
    @Resource
    private SysUserDao sysUserDao;
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String username = principalCollection.getPrimaryPrincipal().toString();
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setRoles(sysUserDao.queryRoleByUserName(username));
        simpleAuthorizationInfo.setStringPermissions(sysUserDao.queryPermitByUserName(username));
        return simpleAuthorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String username = authenticationToken.getPrincipal().toString();
        SysUser sysUser = sysUserDao.querySysUserByUserName(username);
        if (sysUser == null) {
            throw new UnknownAccountException();
        } else {
            AuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(username,sysUser.getPassword(),getName());
            return authenticationInfo;
        }
    }
}
```
>创建配置类
``` java
@Configuration
public class ShiroConfig {
    @Resource
    private MyRealm myRealm;
    @Bean
    public SecurityManager SecurityManager() {
        DefaultWebSecurityManager defaultWebSecurityManager =
                new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(myRealm);
        return defaultWebSecurityManager;
    }
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        shiroFilterFactoryBean.setLoginUrl("/login.html");
        shiroFilterFactoryBean.setUnauthorizedUrl("/norele.html");
        Map<String, String> map = new HashMap<>();
        map.put("/sysUser/login", "anon");
        map.put("/**","authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        return shiroFilterFactoryBean;
    }
    @Bean
    @ConditionalOnMissingBean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator =
                new DefaultAdvisorAutoProxyCreator();
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor =
                new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
}
```
>定制异常处理类
``` java
@ControllerAdvice
public class MyException {
    @ExceptionHandler
    @ResponseBody
    public Result ErrorHandler(Exception e) {
        Result result = new Result();
        if(e instanceof AuthorizationException){
            result.setMessage("抱歉，你的权限不足！");
            result.setCode(500);
        } else if (e instanceof AuthenticationException) {
            result.setMessage("登陆失败！");
            result.setCode(500);
        }
        return result;
    }
}
```
>登陆
``` java
@RequestMapping("login")
public Result login(String username,String password) {
    UsernamePasswordToken token = new UsernamePasswordToken(username,password);
    Subject subject = SecurityUtils.getSubject();
    Result result = new Result();
    subject.login(token);
    result.setCode(200);
    result.setMessage("登陆成功！");
    return result;
}
```
>在方法或类上加上以下注解，用于设置什么角色或权限可以访问
``` java
@RequiresRoles //设置角色
@RequiresPermissions	//设置权限
```
## 使用Ehcache缓存框架实现登录次数锁定
>加入依赖
``` xml
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
    <version>2.6.11</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.5.3</version>
</dependency>
```
>创建缓存框架的配置文件ehcache.xml
``` xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <diskStore path="java.io.tmpdir"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
    <!-- 定义一个缓存的配置 -->
     <!-- timeToIdleSeconds 缓存清理时间 -->
    <cache name="passwordRetryEhcache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="300"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true">
    </cache>
</ehcache>
```
>创建类继承SimpleCredentialsMatcher重写doCredentialsMatch（用于匹配登录）
``` java
public class MyPasswordMatcher extends SimpleCredentialsMatcher {
    private Cache<String, AtomicInteger> retryCount;
    //通过参数传入的缓存管理对象，获取缓存数据
    public MyPasswordMatcher(EhCacheManager ehCacheManager) {
        retryCount = ehCacheManager.getCache("passwordRetryEhcache");
    }

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        String username = token.getPrincipal().toString();
        AtomicInteger count = retryCount.get(username);
        if (count == null) {//第一次登录记录
            count = new AtomicInteger(0);
            retryCount.put(username, count);
        }
        if (count.incrementAndGet() > 3) {//当密码连续错误三次以上
            throw new LockedAccountException();//账号锁定异常
        }
        boolean f = super.doCredentialsMatch(token, info);
        if (f) {//登录验证成功，清空错误次数
            retryCount.remove(username);
        }
        return f;
    }
}
```
>在ShiroConfig中注入
``` java
@Bean
public MyPasswordMatcher myPasswordMatcher(EhCacheManager ehCacheManager) {
    MyPasswordMatcher myPasswordMatcher = new MyPasswordMatcher(ehCacheManager);
    return myPasswordMatcher;
}
@Bean
public EhCacheManager ehCacheManager() {
    EhCacheManager ehCacheManager = new EhCacheManager();
    ehCacheManager.setCacheManagerConfigFile("classpath:ehcache.xml");
    return ehCacheManager;
}
@Bean
public SecurityManager SecurityManager(MyPasswordMatcher myPasswordMatcher) {
    DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
    myRealm.setCredentialsMatcher(myPasswordMatcher);//设置使用自己重写的登录匹配方法
    defaultWebSecurityManager.setRealm(myRealm);
    return defaultWebSecurityManager;
}
```
>拦截异常
``` java
@ControllerAdvice
public class MyException {
    @ExceptionHandler
    @ResponseBody
    public Result ErrorHandler(Exception e) {
        Result result = new Result();
        if(e instanceof LockedAccountException){
            result.setMessage("账号已被锁定，请5分钟之后再试！");
            result.setCode(500);
        } else if(e instanceof AuthorizationException){
            result.setMessage("抱歉，你的权限不足！");
            result.setCode(500);
        } else if (e instanceof AuthenticationException) {
            result.setMessage("登陆失败！");
            result.setCode(500);
        }
        return result;
    }
}
```
