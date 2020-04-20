

![](E:\download\netcore源码剖析.png)



```c#
//ServiceCollection内部是个list<ServiceDescriptor>
var services = new ServiceCollection();
services.AddSingleton<IWebHostEnvironment>(_hostingEnvironment);
//下面是源码，是IServiceCollection的扩展方法
public static IServiceCollection AddSingleton(this IServiceCollection services, Type serviceType, Object implementationInstance)
        {

            if (services == null)
            {
                throw new ArgumentNullException("services");
            }
            if (serviceType == null)
            {
                throw new ArgumentNullException("serviceType");
            }
            if (implementationInstance == 0)
            {
                throw new ArgumentNullException("implementationInstance");
            }
            ServiceDescriptor serviceDescriptor = new ServiceDescriptor(serviceType, implementationInstance);
            services.Add(serviceDescriptor);
            return services;
        }



```

