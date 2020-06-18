# Zuul
## 认识Zull
###  zuul是什么？
Zuul包含了对请求的路由和过滤两个最主要的功能：
其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础.
Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。
 注意：Zuul服务最终还是会注册进Eureka

## 如何使用：
> 项目加入依赖:
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```
>配置文件 application.yml
``` yml
server:
  port: 9000
  
spring:
  application:
    name: zuul
    
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:3000/eureka/
  instance:
    instance-id: zuul-1
    prefer-ip-address: true
```
>配置类加注解
``` java
@EnableZuulProxy
```
>到这里 一个简单的zuul已经搭建好了
 在实际开发当中我们肯定不会是这样通过微服务调用，比如我要调用power 可能只要一个/power就好了 而不是/server-power(服务名)
 在yml加入以下配置即可:
``` yml
zuul:
  #所有的微服务都不可以通过服务名访问 *:所有
  ignored-services: "*"
  routes:
    power:
      serviceId: client-power
      path: /power/**
  prefix: /api #前缀
  #strip-prefix: false #获取请求路径时是否去掉前缀
```
## Zuul过滤器:
过滤器(filter)是zuul的核心组件 zuul大部分功能都是通过过滤器来实现的。 zuul中定义了4种标准过滤器类型，这些过滤器类型对应于请求的典型生命周期。 PRE：这种过滤器在请求被路由之前调用。可利用这种过滤器实现身份验证、在 集群中选择请求的微服务、记录调试信息等。 ROUTING：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服 务的请求，并使用 Apache HttpCIient或 Netfilx Ribbon请求微服务 POST:这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准 的 HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。 ERROR：在其他阶段发生错误时执行该过滤器。 

如果要编写一个过滤器，则需继承ZuulFilter类 实现其中方法:
``` java
@Component
public class LogFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;//过滤器类型
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER+1;//过滤器优先级
    }

    @Override
    public boolean shouldFilter() {
        return true;//是否使用当前过滤器
    }

    @Override//过滤方法
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        System.out.println(ctx.get(FilterConstants.REQUEST_URI_KEY));
        System.out.println(request.getRemoteAddr()+"访问了"+request.getRequestURI());
        return null;
    }
}
```
## Zuul容错与回退
>zuul默认是整合了hystrix和ribbon的， 提供降级回退，那如何来使用hystrix呢？
>
>我们自行写一个类，继承FallbackProvider 类 然后重写里面的方法
``` java
@Component
public class FallBackProvider implements FallbackProvider {
    @Override
    public String getRoute() {
        return "client-power";
    }

    @Override                                  //出错微服务名    //异常
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        if(cause instanceof HystrixTimeoutException){//超时异常
            return response(HttpStatus.GATEWAY_TIMEOUT);
        }else{
            return response(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    private ClientHttpResponse response(final HttpStatus status) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return status;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return status.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return status.getReasonPhrase();
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("系统繁忙，稍后再试".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}

```
