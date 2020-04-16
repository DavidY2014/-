# Framework MVC5 (一)

## 1，狭义mvc是个遵循三层架构的web框架

- view，用户视图
- controllers，决定用户使用哪个视图，且包含逻辑运算
- models，模型，承载业务数据

（1）ActionResult是ViewResult的基类

```
public JosnResult GetJson(){
	return new JsonResult(){
		Data = new {
			Id = id;
			Name =name;
		}
	}
}

public FileResult GetFile(){
	return File("filename");
}
```

（2）MVC可以返回html，string，json，xml，file，图片

（3）WebAPI和MVC的区别

- 不管aspx,ashx,webapi.mvc 都是使用http协议，所有请求都可以处理
- aspx：比较重，默认有页面的生命周期---前后端柔和，viewstate和cs对应绑定
- ashx：属于轻量级，没有页面概念
- MVC：前后分离，可以指定任意视图，可以一套后台，多套UI

```
public ActionResult Index(){
	return View("~/Views/First/index1.html")
}
```

- WebApi：专人做专事，使用Restful风格，netcore二者又融合进管道了

## 2，IIS部署，Global，Log4

（1）Global文件

Appliacation_start() ：全局启动是执行的，适合做初始化的工作

## 3，数据传值的多种方式

（1）页面传值

- viewbag 和 viewdata,Tempdata

```
viewbag.XX = "AA";  viewbag["XX"]="AA";
viewdata.prop = "AA";viewdata["prop"] = "AA";
```

- viewData字典传值，里面事object，需要类型转换

- viewbag是dynamic传值，可以随便属性访问，运行时检测

- viewData和viewbag两者会先后覆盖

- tempdata是独立存储，不会覆盖,临时数据，存储在session中，可以跨action传递

- model 适合复杂数据的传递

- 工程的view->web.config是视图配置，所有的视图cshtml的父类都是system.web.mvc.webviewpage

- MasterPage-->Layout ，是多个页面中不变的部分

  ```
  Layout = "~/Views/Shared/_Layout.cshtml";
  ```



## 4，Route使用和扩展，Area

```
protected void Application_Start()
{
    AreaRegistration.RegisterAllAreas();//注册区域
    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);//注册过滤器
    RouteConfig.RegisterRoutes(RouteTable.Routes);//注册路由
    BundleConfig.RegisterBundles(BundleTable.Bundles);
}
```

1，控制器类可以出现在MVC项目之外，只要继承Controller就行

## 5，Razor语法，前后端语法混编

（1）html+后端代码混编



## 6，Html扩展控件，后端封装前端

```
@Html.Br()
@Html.TextBox("")
```

通过扩展控件可以让html页面输出完整，由后端进行控制,但是UI改动需要重新发布，不符合前后端分离的开发方式

## 7，模板页Layout，局部页PartialView

（1）相当于就是把多个js打包成一个js文件进行引用

```
BundleConfig.RegisterBundles(BundleTable.Bundles);//打包工具
//把include的多个js文件打包成一个js
bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                        "~/Scripts/jquery-{version}.js"));

bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
                        "~/Scripts/jquery.validate*"));
                        
//使用
@Style.Render("~/bundles/jquery");
//就是页面的结合点
@RenderBody();
```

## 8，IOC和MVC的结合，工厂的创建和Bussiness初始化

（1）管理方案的Nuget可以全局生效,避免单独工程引用的不一致性

（2）MVC使用EF6：引入EF依赖+数据库连接配置

（3）mvc请求->路由扫描->找到controllers->实例化controller（其中实例化是通过IOC容器实现的）

定义自定义的controllerFactory，返回自定义的IOC容器

```c#
//下面是system.web.routing 框架的源码		
protected virtual void ProcessRequest(HttpContextBase httpContext)
		{
			RouteData routeData = this.RouteCollection.GetRouteData(httpContext);
			if (routeData == null)
			{
				throw new HttpException(404, SR.GetString("UrlRoutingHandler_NoRouteMatches"));
			}
			IRouteHandler routeHandler = routeData.RouteHandler;
			if (routeHandler == null)
			{
				throw new InvalidOperationException(SR.GetString("UrlRoutingModule_NoRouteHandler"));
			}
			RequestContext requestContext = new RequestContext(httpContext, routeData);
			IHttpHandler httpHandler = routeHandler.GetHttpHandler(requestContext);
			if (httpHandler == null)
			{
				throw new InvalidOperationException(string.Format(CultureInfo.CurrentUICulture, SR.GetString("UrlRoutingModule_NoHttpHandler"), new object[]
				{
					routeHandler.GetType()
				}));
			}
			this.VerifyAndProcessRequest(httpHandler, httpContext);
		}
/******************************************************************/
/// <summary>Selects the controller that will handle an HTTP request.</summary>
	public class MvcHandler : IHttpAsyncHandler, IHttpHandler, IRequiresSessionState
	{
		private struct ProcessRequestState
		{
			internal IAsyncController AsyncController;

			internal IControllerFactory Factory;

			internal RequestContext RequestContext;

			internal void ReleaseController()
			{
				this.Factory.ReleaseController(this.AsyncController);
			}
		}
/************************************************************************/
        /// <summary>Processes the request by using the specified base HTTP request context.</summary>
		/// <param name="httpContext">The HTTP context.</param>
		protected internal virtual void ProcessRequest(HttpContextBase httpContext)
		{
			IController controller;
			IControllerFactory controllerFactory;
			this.ProcessRequestInit(httpContext, out controller, out controllerFactory);
			try
			{
				controller.Execute(this.RequestContext);
			}
			finally
			{
				controllerFactory.ReleaseController(controller);
			}
		}
/*************************************************************************/
        		private void ProcessRequestInit(HttpContextBase httpContext, out IController controller, out IControllerFactory factory)
		{
			HttpContext current = HttpContext.Current;
			if (current != null)
			{
				bool? flag = ValidationUtility.IsValidationEnabled(current);
				bool flag2 = true;
				if (flag.GetValueOrDefault() == flag2 & flag.HasValue)
				{
					ValidationUtility.EnableDynamicValidation(current);
				}
			}
			this.AddVersionHeader(httpContext);
			this.RemoveOptionalRoutingParameters();
			string requiredString = this.RequestContext.RouteData.GetRequiredString("controller");
			factory = this.ControllerBuilder.GetControllerFactory();
			controller = factory.CreateController(this.RequestContext, requiredString);
			if (controller == null)
			{
				throw new InvalidOperationException(string.Format(CultureInfo.CurrentCulture, MvcResources.ControllerBuilder_FactoryReturnedNull, new object[]
				{
					factory.GetType(),
					requiredString
				}));
			}
		}
```



## 9，WCF搜索引擎的封装调用和AOP的整合

```
//表单提交
searchString = base.HttpContext.Request.Form["serachString"];
//分页查询
PageResult和StaticPagedList
```



## 10，HTTP请求的本质，各种ActionResult扩展订制





## 11，增删改，Ajax删除，表单提交，列表和三级联动





## 12，HttpGet，HttpPost ，BindChildActionOnly特性解读

（1）[ValidateAntiForeryToken]防重复提交，再cookie里加一个key，提交时先校验这个







## 13，用户登录，退出功能实现



## 14，asp.net 对象

（1）HttpContext, HttpRequest, HttpResponse。HttpRuntime，HttpServerUtility

（2）HttpRequest的实例包含了所有来自客户端的所有数据

```c#
下面是httpRequst 的属性
// 获取服务器上 ASP.NET 应用程序的虚拟应用程序根路径。
public string ApplicationPath { get; }

// 获取应用程序根的虚拟路径，并通过对应用程序根使用波形符 (~) 表示法（例如，以“~/page.aspx”的形式）使该路径成为相对路径。
public string AppRelativeCurrentExecutionFilePath { get; }

// 获取或设置有关正在请求的客户端的浏览器功能的信息。
public HttpBrowserCapabilities Browser { get; set; }

// 获取客户端发送的 cookie 的集合。
public HttpCookieCollection Cookies { get; }

// 获取当前请求的虚拟路径。
public string FilePath { get; }

// 获取采用多部分 MIME 格式的由客户端上载的文件的集合。
public HttpFileCollection Files { get; }

// 获取或设置在读取当前输入流时要使用的筛选器。
public Stream Filter { get; set; }

// 获取窗体变量集合。
public NameValueCollection Form { get; }

// 获取 HTTP 头集合。
public NameValueCollection Headers { get; }

// 获取客户端使用的 HTTP 数据传输方法（如 GET、POST 或 HEAD）。
public string HttpMethod { get; }

// 获取传入的 HTTP 实体主体的内容。
public Stream InputStream { get; }

// 获取一个值，该值指示是否验证了请求。
public bool IsAuthenticated { get; }

// 获取当前请求的虚拟路径。
public string Path { get; }

// 获取 HTTP 查询字符串变量集合。
public NameValueCollection QueryString { get; }

// 获取当前请求的原始 URL。
public string RawUrl { get; }

// 获取有关当前请求的 URL 的信息。
public Uri Url { get; }

// 从 QueryString、Form、Cookies 或 ServerVariables 集合中获取指定的对象。
public string this[string key] { get; }

// 将指定的虚拟路径映射到物理路径。
// 参数:  virtualPath:  当前请求的虚拟路径（绝对路径或相对路径）。
// 返回结果:  由 virtualPath 指定的服务器物理路径。
public string MapPath(string virtualPath);
```