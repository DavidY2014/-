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









