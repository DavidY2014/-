**1，aspnetcore的启动顺序**

```
>ConfigureWebHostDefaults
>ConfigureHostConfiguration
>ConfigureAppConfiguration
>ConfigureServices,ConfigureLogging,Startup,Startup.ConfigureServices
>Startip.Configure
```

**2，依赖注入的核心类型**

```
>IServiceCollection
>ServiceDescriptor
>IServieProvider
>IServiceScope
```

**3，生命周期**

```
单例，作用域，瞬时(通过显示hashcode来判断)
单例：是程序运行期间只产生一个对象，每次请求都是一样
作用域Scope:每次请求不一样
瞬时：每次调用不一样
```

**4，容器注入**

```
public interface IServiceProviderFactory<TContainerBuilder>
>引入autoface
Autofac.Extensions.DependencyInjection
Autofac.Extras.DynamicProxy

```

```
//引入autofac容器
public static IHostBuilder CreateHostBuilder(string[] args) =>
	Host.CreateDefaultBuilder(args)
    .UseServiceProviderFactory(new AutofacServiceProviderFactory())
    .ConfigureWebHostDefaults(webBuilder =>
    {
        webBuilder.UseStartup<Startup>();
    });
```

**5，aop**

```
//创建拦截器,实现IInterceptor接口
public class MyInterceptor : IInterceptor
{
	public void Intercept(IInvocation invocation)
    {
        Console.WriteLine($"Intercepter before,Method:{invocation.Method.Name}");
        invocation.Proceed();
         Console.WriteLine($"Intercepter after,Method:{invocation.Method.Name}");
        }
    }
 }
 //启用拦截器
  builder.RegisterType<MyInterceptor>();
  builder.RegisterType<MyService>().As<IService>()
  .PropertiesAutowired().InterceptedBy(typeof(MyInterceptor));

```

**6，配置**

```
//核心组件包
>Microsoft.Extensions.Configuration.Abstractions
>Microsoft.Extensions.Configuration
//命令行配置
通过launchSetting.json来配置

```

**7，配置跟踪**

```
IChangeToken IConfiguration.GetReloadToken()
//扩展方法的作用是屏蔽具体的实现类功能，通过扩展方法去屏蔽细节

```

**8，编译过程**

```
>c#/VB project
>编译生成IL+metadata
>JIT结合BCL基础类库编译生成机器码
//CLR 运行时环境
//Framework：框架，包含CLR。编译器，BCL
>IIS需要安装dotnet-hosting+dotnet-sdk
```

**9，MVC**

```
>TempData的使用首先需要再config中注册services.AddSession()才能使用

```

**10，Core的IIS部署**

```
>core的配置文件读取
Configuration注入进来的类实现，依赖于Configuration使用xpath来访问appsetting.json的值

```

**10，Core的Log4net**

```c#
//Log4Net的集合,nuget
//Microsoft.Extensions.Logging.Log4Net.AspNet//把log4net整合进netcore的扩展包
//Log4net

public static IHostBuilder CreateHostBuilder(string[] args) =>
   Host.CreateDefaultBuilder(args)
   .ConfigureLogging((context,LoggingBuilder)=> {
         LoggingBuilder.AddFilter("System", LogLevel.Warning);
         LoggingBuilder.AddFilter("Microsoft", LogLevel.Warning);
         LoggingBuilder.AddLog4Net();
    })

 public void Configure(IApplicationBuilder app, IWebHostEnvironment env
            ,ILoggerFactory factory){
    //因为log4net已经注入到IOC了，所以直接通过使用
}

```

**11，ServiceCollection的使用**

```c#
IServiceCollection container = new ServiceCollection();
container.AddTransient<IService>();//扩展方法
ServiceProvider provider =  container.BuildServiceProvider();
provider.GetService<IService>();
    
```

**12，三方DI容器Autofac+AOP**

```c#
//autofac+autofac.extension.dependencyinjection
//AutofacXXXDynamicProxy

public void ConfigureContainer(ContainerBuilder builder)
{
    builder.RegisterType<MyService>().As<IService>();
    builder.RegisterType<MyRecordInterceptor>();
    builder.RegisterType<MyService>().As<IService>().EnableInterfaceInterceptors().SingleInstance();
}


[Intercept(typeof(MyRecordInterceptor))]
public class MyService:IService
{

   public void Show()
   {
        Console.WriteLine("MyService:Show");
    }
}

public class MyRecordInterceptor : IInterceptor
{
    public void Intercept(IInvocation invocation)
    {
         Console.WriteLine($"Intercepter before,Method:{invocation.Method.Name}");
         invocation.Proceed();
          Console.WriteLine($"Intercepter after,Method:{invocation.Method.Name}");
     }
 }

```

**13，Filter调用顺序,ActionFilter等**

```
Filter内部实现的方法OnActionExecuting 和 OnActionExecuted
多个filter的调用顺序
//ResourcewFilter



```

**14，autofac的文件配置**



**15，Middleware，三种注册和实现，新管道模型解析**

```

```

**16，权限认证**

（1）登录时候写入session

（2）在需要权限控制方法上标记一个权限attribute，并在attribute中对session进行判断

（3）actionFilter可以达到类似效果，在基类中通过attribute进行认证，但是局限性比较大

（4）netcore中通过中间件进行认证

```c#
app.Map("/login", builder => builder.Use(next =>
{
     return async (context) =>
     {
           var claimIdentity = new ClaimsIdentity();
           claimIdentity.AddClaim(new Claim(ClaimTypes.Name, "TestUser"));
           await context.SignInAsync("",new ClaimsPrincipal(claimIdentity));
           await context.Response.WriteAsync("login successed");
      };
}));
```







**17，认证和授权原理机制**





**18，用户登录权限验证，基于cookie的认证和授权**



**19，进程内托管和进程外托管**

（1）传统的netframework是进程内托管，Application运行在IIS的w3wp.exe进程中

（2）netcore是进程外托管，dotnet.exe自己启动一个kestrel服务进程来运行Application，当然netcore程序也支持进程内托管，如下配置

**20，JWT**

（1）API保护有哪些保护方式：

非登录：报文sha+密钥+加密算法，header传递解密

登录：Session+cookie，依靠浏览器，有局限性，如果单页面登陆，前后端分离的话会丢失

登录JWT：

（2）通过dotnet watch run 可以监听文件变更，进行调试

**21，netcore框架代码的核心**

```
webhostbuilder类

```

**22，生命周期的测试**

```
//构造函数注入
private readonly IService _service
ctor(IService service){
	_service = service
}
//如果是scope，下面方法只进入一次构造函数，transient会进入两次构造函数
_service.show();
_service.show();

```

**23，AOP**

（1）AOP的运用场景

日志，事务，缓存，异常处理，性能优化，这些不是业务逻辑的功能，最好不要使用到OOP中

问题：在通常的开发中，需要实现日志，事务的处理，我们一般会强写到业务代码中，为了解决这种问题最好通过AOP方式实现，oop方式会让整个代码复杂啰嗦，不可控。

（2）AOP，中间件UseAuthentication，过滤器AuthorizationFilter思想都是AOP，但是拦截的东西和颗粒度不同

**24，DTO**

（1）充血模型，就是除了基本的属性外还包含各种方法

**25，Automapper**



**26，跨域**

（1）没有同源策略的危害：

防止恶性请求，比如恶意站点可以获取其他站点保存的信息

（2）Ajax的请求步骤

> 创建XMLHttpRequest对象

> 使用open方法设置请求参数，open(method,url,是否异步)

> 发送请求

> 注册事件，注册onreadystatechange事件

> 获取返回数据，更新UI

（3）JSONP：

通过script标签不受同源策略限制可以发送跨域请求

（4）Proxy：



**27，Win部署**

（1）SCD，独立发布部署，不需要框架依赖,放到一台没有安装sdk和runtime的环境进行部署

（2）部署成service，使用nssm软件

```
>nssm install
```

（3）IIS托管，爆出的错误一般是windows hosting 的未安装

**28，Linux部署**

（1）安装https://docs.microsoft.com/zh-cn/dotnet/core/install/linux-package-manager-centos7

（2）运行

（3）守护进程：PM2或linux自带的守护进程

可以用Supervisor 实现进程守护

https://www.cnblogs.com/wyt007/p/8288929.html

```c#
//安装supervisor
yum install supervisor 
//启动服务
supervisord -c /etc/supervisord.conf
//进入 /etc目录找到supervisord.conf 配置文件 和 supervisord.d 文件夹，并编辑
[root@VM_0_8_centos etc]# cd supervisord.d/
[root@VM_0_8_centos supervisord.d]# ls
[root@VM_0_8_centos supervisord.d]# vim netcoredeploy.ini
{
    [program:DeployLinux]   #DeployLinux  为程序的名称
    command=dotnet DeployLinux.dll #需要执行的命令
    directory=/home/publish #命令执行的目录
    environment=ASPNETCORE__ENVIRONMENT=Production #环境变量
    user=root #用户
    stopsignal=INT 
    autostart=true #是否自启动
    autorestart=true #是否自动重启
    startsecs=3 #自动重启时间间隔（s）
    stderr_logfile=/var/log/ossoffical.err.log #错误日志文件
    stdout_logfile=/var/log/ossoffical.out.log #输出日志文件
}

supervisorctl reload  //重新加载配置文件
//如果遇到问题了直接重新安装
如果是独立发布成linux64位的话，需要chmod+x  XXXX程序，不是dll
root@VM_0_8_centos Linux64_IndepentPub]# chmod +x BangBangFuli.API.MVCDotnet2
[root@VM_0_8_centos Linux64_IndepentPub]# ./BangBangFuli.API.MVCDotnet2 

```

（4）坑就是linux可能无法访问sqlserver数据库

（5）nginx的安装，反向代理

（6）centos无法访问sqlserver的问题

https://www.cnblogs.com/xiaxiaolu/p/10309064.html

sqlserver版本太低导致的

https://blog.csdn.net/finn_wft/article/details/89148394  nginx代理

（7）通过在数据库管理工具上执行show @@version 可以查看sqlserver的版本号

目前在windows上sql.client.dll可以访问，linux上需要高版本的sqlserver服务才行

**29，IdentityServer**

（1）一个身份验证框架，可以实现多个项目的集中认证和令牌颁发

（2）但是一些简单的项目用jwt就可以了，如果想要把多个功能模块分散处理，提高安全性的话可以考虑

（3）jwt和OAuth2.0协议：

JWT是个认证框架实现，OAuth2.0是个授权协议，是规范不是实现

（4）一个客户端跳转到授权服务器嘛，然后登录了再跳转过来，增加了个用户角色授权

比如项目地址为IP:1234  ，id4的为IP2：5002，那么在项目登陆界面点击授权登陆时会跳转到5002的授权界面，授权登陆成功后会执行回调地址到IP:1234/index,此时完成登陆操作，无需进行账号密码的登陆

**30，索引器**

```c#
public class IDXer
    {
        private string[] name=new string[10];

        //索引器必须以this关键字定义，其实这个this就是类实例化之后的对象
        public string this[int index]
        {
            get 
            {
                return name[index];
            }
            set
            {
                name[index] = value;
            }
        }  
    }

 public class Program
    {
        static void Main(string[] args)
        {
            //最简单索引器的使用           
            IDXer indexer = new IDXer();
            //“=”号右边对索引器赋值，其实就是调用其set方法
            indexer[0] = "张三";
            indexer[1] = "李四";
            //输出索引器的值，其实就是调用其get方法
            Console.WriteLine(indexer[0]);
            Console.WriteLine(indexer[1]);
            Console.ReadKey();
        }
    }
        
```

# 缓存专题

#### **1，总览**

客户端->反向代理->本地->分布式缓存

Aspnetcore的缓存机制

#### **2，缓存全景图**

（1）浏览器---客户端缓存，通过F12可以看出来

（2）DNS----CDN缓存

（3）反向代理----负载均衡

（4）服务器-----本地缓存，DB

（5）还有分布式缓存

![image-20200420213012804](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200420213012804.png)

浏览器缓存：

![image-20200420213147674](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200420213147674.png)

#### 3，http协议缓存

responseheader

（1）是否缓存Expires ，Cache-Control

（2）缓存时间，Etag，Last-Modified

```
public IActionResult Privacy()
{
	base.HttpContext.Response.Headers["Cache-Control"]="public,max-age=600";
}
```

![image-20200421103503472](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421103503472.png)

（3）etag：response返回304，代表从缓存重读取，资源未更新

#### 4，CDN缓存

![image-20200421105339077](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421105339077.png)

（1）CDN和反向代理的关系

cdn是正向代理

#### 5，IIS缓存

（1）在conf中进行配置

#### 6，服务端缓存

（1）nuget MemoryCache

（2）引入IMemoryCache

（3）集群部署：通过netcore的入参启动不同端口号服务进程，一个程序启动多个进程，然后再nginx中配置

![image-20200421123131764](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421123131764.png)



![image-20200421123559876](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421123559876.png)



（4）分布式缓存是redis

![image-20200421133327220](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421133327220.png)



#### 7，中间件（Middleware）和过滤器（Filter）的区别

源码分析

![image-20200421141416837](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421141416837.png)



（1）基于IResouceFilter 扩展定制缓存实现，对Action进行了拦截

#### 8，中间件缓存

（1）services.AddResponseCaching()

（2）app.useResponseCaching()

![image-20200421142930006](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421142930006.png)

#### 9，缓存穿透，缓存击穿，缓存雪崩

参考https://blog.csdn.net/kongtiao5/article/details/82771694

https://www.jianshu.com/p/d00348a9eb3b

（1）缓存穿透：直接把null存起来，或者直接接口进行过滤。

（2）缓存击穿：某个值突然过期，此时大量请求突然涌入数据库，导致数据库崩掉

让热点数据不过期。

（3）缓存雪崩：大量的key同时失效，数据库压力太大，导致db挂掉

随机过期时间，避免同时过期，或者不过期只更新，分布式数据均衡分配

# Net5入门

![image-20200421181529215](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421181529215.png)

#### 1，nginx配置

![image-20200421181848866](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421181848866.png)



（1）集群，负载均衡，在nginx中配置weight

（3）session的解决方案：

StateServer,Sqlserver,Redis，cookie验证，token（JWT，IDS4）

#### 2，根据ISession的衍生

（1）接口最小化涉及

（2）组件化开发，通过扩展方法--减小体积

（3）IOC---只依赖抽象，不依赖于细节



#### 3，Aspnet的管道模型

![image-20200421192139819](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421192139819.png)



（1）基本流程是通过事件进行处理

比如在框架内部会一一执行事件，事件注册的话，就执行注册成功的委托，相当于是一一执行事件函数

![image-20200421192722691](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421192722691.png)

![image-20200421192804773](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421192804773.png)

![image-20200421192823632](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421192823632.png)

（2）把所有环节固定执行，通过事件完成扩展，这套涉及思想已经陈旧了，而且会导致框架庞大臃肿

![image-20200421193649445](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421193649445.png)

（3）Redis注册,做分布式session

```c#
public void ConfigureService(IServiceCollection services)
{
	services.AddDistributedRedisCache(
		options => {
            options.Configuration = "127.0.0.1:6379";
			options.InstanceName = "Redistest";
		}
	);
}
```

（4）中间件的写法

![image-20200421204131409](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200421204131409.png)

（5）中间件的作用

日志监控，缓存，针对http请求，黑白名单，鉴权

（6）不同层级AOP

下面是不同filter的执行顺序

![image-20200422100729509](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200422100729509.png)

过滤器作用域设置是非常灵活的，可以选择为：

1. 全局有效（*整个 MVC 应用程序下的每一个 Action*）；
2. 仅对某些 Controller 有效 (*控制器内所有的 Action* )；
3. 仅对某些 Action 有效;

#### Authorization Filter

授权过滤器在过滤器管道中第一个被执行，通常用于验证请求的合法性（*通过实现接口 IAuthorizationFilter or IAsyncAuthorizationFilter*）

#### Resource Filter

资源过滤器在过滤器管道中第二个被执行，通常用于请求结果的缓存和短路过滤器管道（*通过实现接口 IResourceFilter or IAsyncResourceFilter*）

#### Action Filter

Acioin 过滤器可设置在调用 Acioin 方法之前和之后执行代码，如：请求参数的验证（*通过实现接口 IActionFilter or IAsyncActionFilter*）

#### Exception Filter

程序异常信息处理（*通过实现接口 IExceptionFilter or IAsyncExceptionFilter*）

#### Result Filter

Action 执行完成后执行，对执行结果格式化处理（*通过实现接口 IResultFilter or IAsyncResultFilter*）

```c#
public class CustomExceptionFilterAttribute : ExceptionFilterAttribute
{
  public override void OnException(ExceptionContext context)
  {
    context.Result = new ObjectResult(new ApiResult()
    {
      Code = Enum.ResultCode.Exception,
      ErrorMessage = $"CustomExceptionFilterAttribute: {context.Exception.Message}"
    });
  }
}
```

```c#
[CustomExceptionFilter]
public ApiResult ExceptionAttributeTest()
{
  throw new Exception("Boom");
}
```

参考https://www.jianshu.com/p/c7c7b6bf79f7

https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-2.1

https://www.cnblogs.com/dotNETCoreSG/p/aspnetcore-4_4_3-filters.html

*只能* 实现一个过滤器接口，要么是同步版本的，要么是异步版本的，*鱼和熊掌不可兼得* 。如果你需要在接口中执行异步工作，那么就去实现异步接口。否则应该实现同步版本的接口。框架会首先检查是不是实现了异步接口，如果实现了异步接口，那么将调用它。不然则调用同步接口的方法。如果一个类中实现了两个接口，那么只有异步方法会被调用。最后，不管 action 是同步的还是异步的，过滤器的同步或是异步是独立于 action 的。

## 过滤器作用域

过滤器具有三种不同级别的 *作用域* 。你可以在特定的 action 上用特性（Attribute）的方式使用特定的过滤器；也可以在控制器上用特性的方式使用过滤器，这样就会将效果应用在控制器内所有的 action 上；或者注册一个全局过滤器，它将作用于整个 MVC 应用程序下的每一个 action。

如果想要使用全局过滤器的话，在你配置 MVC 的时候在 `Startup` 的 `ConfigureServices` 方法中添加：

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        options.Filters.Add(typeof(SampleActionFilter)); // by type
        options.Filters.Add(new SampleGlobalActionFilter()); // an instance
    });

    services.AddScoped<AddHeaderFilterWithDi>();
}
```

熟悉MVC框架的同学应该知道，MVC也提供了5大过滤器供我们用来处理请求前后需要执行的代码。分别是`AuthenticationFilter`,`AuthorizationFilter`,`ActionFilter`,`ExceptionFilter`,`ResultFilter`。

根据描述，可以看出中间件和过滤器的功能类似，那么他们有什么区别？为什么又要搞一个中间件呢？
其实，过滤器和中间件他们的关注点是不一样的，也就是说职责不一样，干的事情就不一样。

> 举个栗子，中间件像是`埃辛诺斯战刃`，过滤器像是`巨龙之怒，泰蕾苟萨的寄魂杖` ,你一个战士拿着`巨龙之怒，泰蕾苟萨的寄魂杖`去战场杀人，虽然都有伤害，但是你拿着法杖伤害低不说，还减属性啊。

同作为两个AOP利器，过滤器更贴合业务，它关注于应用程序本身，比如你看`ActionFilter` 和 `ResultFilter`，它都直接和你的Action，ActionResult交互了，是不是离你很近的感觉，那我有一些比如对我的输出结果进行格式化啦，对我的请求的ViewModel进行数据验证啦，肯定就是用Filter无疑了。它是MVC的一部分，它可以拦截到你Action上下文的一些信息，而中间件是没有这个能力的。

那么，何时使用中间件呢？我的理解是在我们的应用程序当中和业务关系不大的一些需要在管道中做的事情可以使用，比如**身份验证，Session存储，日志记录**等。其实我们的 asp.net core项目中本身已经包含了很多个中间件。

（7）代码分层--业务下沉-----通过依赖注入使用对象---使用aop

如果想要更细化的aop控制，考虑使用autofac，控制力度可以到方法粒度

![image-20200422120000490](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200422120000490.png)

（8）IOC&AOP对业务层进行操作，可以缓存数据，请求要进入action-view

一般用来降低数据库压力，减少数据查询

（9）想缓存数据，但是不进入action，用actionfilter或resourcefilter--进入mvc中

（10）可以再middleware拦截，完全不进入mvc---responsecachemiddleware

（11）AOP的实现方式，解决程序扩展问题

静态实现：代理模式&装饰器模式

动态实现：dynamic proxy

常用的---FIlter，IOC容器扩展autofac

# Asp.netcore3.1 

1，最小化接口设计，扩展方法实现组件化开发，比如日志组件log4net的源码分析

![image-20200422135658441](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200422135658441.png)

2，kestrel是一个简化版的IIS web服务，负责监听请求，转发到代码进行处理

3，http请求管道模型---就是http请求被处理的步骤--http请求是一段协议文本，被Kestrek解析得到httpcontext

---然后被后台改代码处理Request----返回Response---经由Kestrel回发到客户端

所谓管道就是拿着httpcontext，经过多个步骤的加工，生成response

4，中间件代码

```c#
 Func<RequestDelegate, RequestDelegate> middleware1 = (ctx) => {
                //对传进来的context ，ctx进行处理

                return new RequestDelegate(async ctx=> {
                    await Task.CompletedTask;
                });

     			//两种写法都可以，lambda会自动的被包装成一个委托，本质上lambda是一个函数，只不过编译器会隐式的包装成一个委托
                //return  async ctx =>
                //{
                //    await Task.CompletedTask;
                //};
            };

```

5，每一次请求，都会执行一次管道操作

![image-20200422193457352](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200422193457352.png)

```c#
  #region 自定义middleware

            app.Use(next =>
            {
                return async ctx =>
                {
                    Console.WriteLine($"Middleware 1 start {DateTime.Now}\n");
                    await next.Invoke(ctx);
                    Console.WriteLine($"Middleware 1 end {DateTime.Now}\n");
                };
            });

            app.Use(next =>
            {
                return async ctx =>
                {
                    Console.WriteLine($"Middleware 2 start {DateTime.Now}\n");
                    await next.Invoke(ctx);
                    Console.WriteLine($"Middleware 2 end {DateTime.Now}\n");
                };
            });

            app.Use(next =>
            {
                return async ctx =>
                {
                    Console.WriteLine($"Middleware 3 start {DateTime.Now}\n");
                    await next.Invoke(ctx);
                    Console.WriteLine($"Middleware 3 end {DateTime.Now}\n");
                };
            });

            #endregion

```

![image-20200422215009540](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200422215009540.png)



#### 6，控制台调试

IIS托管---w3wp

控制器命令行---dotnet

#### 7，Filter构造函数导入的问题

![image-20200423114121645](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423114121645.png)

（1）结论：特性的构造函数只能是常量不能是变量

（2）通过ServiceFilter来构造，其实是个工厂方法

```c#
[ServiceFilter(typeof(CustomExceptionFilterAttribute))]
        public IActionResult ConsoleIndex()
        {
            return View();
        }

```

![image-20200423115446687](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423115446687.png)

（3）Filter执行顺序

![image-20200423131525394](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423131525394.png)

![image-20200423151415273](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423151415273.png)

![image-20200423152922537](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423152922537.png)

上面这个会导致数据缓存到浏览器端

#### 8，用户登录退出，cookie和session验证

（1）不把验证信息写入cookie和session，写入Claim中

参考 https://www.cnblogs.com/Ivan-Wu/p/10711288.html

#### 9，鉴权UseAuthentication和授权UseAuthentization

（1）认证和授权







#### 10，EF_codeFirst



#### 11，分层封装，基本架构



# 微服务架构

1，分布式架构：

独立部署开发，单体是调用方法BLL---DAL

分布式是调用服务

**和SOA的区别**

![image-20200423211706410](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423211706410.png)



SOA是把多个服务整合起来进行管理，微服务是分拆进行服务

2，微服务架构通信

（1）redis/DB/Queue/硬盘文件：被动通讯

（2）通过服务进行通讯：webservice，wcf，webapi，ashx，aspx：主动触发，数据序列化传输，跨平台，跨语言，http穿透防火墙

（3）GRPC，基于HTTP2协议设计

3，微服务架构设计

![image-20200423214809577](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423214809577.png)

4，启动一个webapi工程，然后从另一个mvc页面发送请求到webapi项目数据，此时会发生跨域问题

跨域问题是浏览器做了限制，但是真实的数据还是返回过来了

![image-20200424002252919](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424002252919.png)

![image-20200424002323089](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424002323089.png)

（1）高可用系统通过集群来实现

![image-20200424002638104](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424002638104.png)

带来的问题是启动多个实例后怎么进行维护和控制

```c#
#region 支持命令行参数启动

            var config = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddCommandLine(args)
                .Build();

#endregion
```

