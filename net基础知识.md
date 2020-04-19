## Q：如何对数组的元素进行倒序排列？

A：xx.Sort()后xx.Reverse()。

## Q：单点登录常用的实现方法有？

A：

1. 存放用户凭证在Cookie中，但是依赖Cookie不太安全而且无法跨域
2. 使用jsonp，和cookie差不多，虽然能解决跨域但加密算法泄露还是不安全
3. token，现在比较常见的方式，通过反复重定向来验证，因为所有信息都存在服务端，所以即使知道了加密算法也无法登录，比较安全。但是项目调用比较复杂。

## Q：在ASP.NET页面之间传递值除了Session、Cookie、Application，还有什么其他方法？

A：QueryString,POST,ViewData(基于字典),ViewBag(基于Dynamic)

## Q：什么是SQL注入，如何防止SQL注入攻击？

A：代码没有过滤特殊字符时，在执行sql语句处填入';等之类的字符来截断sql语句后再输入要执行的sql语句（删表等）来对数据库进行攻击。可以使用SQLParameter(把所有参数当字符串而不是关键字来处理)或者ORM来避免。

## Q：如何在所有页面加载时输出内容？（不懂）

A：使用HTTPModule/中间件或者Filter。

## Q：什么是串行化？

A：将数据使用BinaryFormat从Stream保存到二进制文件，这种文件大小都是2的X次方。

## Q：什么叫应用程序域？

A：就是程序之间的边界，可以理解一个程序是一个应用程序域。

## Q：string str = null和string str = “”的区别？

A：null表示不存在于内存中没有地址，“”表示存在于内存中的“”这个对象

## Q：描述class和struct异同？

A：class是引用类型放在堆上的，struct是值类型放在栈上的。

## Q：try里有一个return语句，那么finally中的代码会不会执行，在return前还是return后？

A：会，在return前执行。看反编译会发现finally的代码放在了函数出口之前

## Q：两个对象值相同x.equals(y)==true，但可能有不同的hash code，这个说法对吗？

A：==比较地址，equals比较值*（等比地址Equals值）*。两个对象值相同则表示拥有相同的HashCode。

## Q：请指出GAC的含义？

A：（Global Assembly Cache）全局程序集缓存。应该是指常用到的dll（比如System.Data,System.Collections）会缓存在windows一个指定的文件夹下，别的程序再使用的话就直接调用，不用重新加载了

## Q：.NET中的new关键字和override关键字的异同？

A：要重写的方法都需要virtual关键字，但是override是重写了父类的方法，调用的话是子类重写后的方法；new是重新写了一个同名的方法，隐藏了父类的方法。**如果把子类强制转换为父类的话new调用还是父类的方法，而override则调用子类重写的方法。**

## Q：后台接口如何鉴权？比如开发了一个后台管理系统，如何防止别人拿到你请求的接口恶意攻击？

A：OAuth使用accessToken访问。或者使用JWT

## Q：纯webapi项目，前端请求的用户信息存储在哪里？

A：web的话localStorage。不同端有不同的存储方式。

## Q：多线程的线程同步如何实现？

A：

1. 使用Monitor/lock关键字创建临界区

2. 读写锁

   ```
   static private ReaderWriterLock _rwlock = new ReaderWriterLock(); 
   // 请求读锁，如果10ms超时退出 
   _rwlock.AcquireReaderLock(10); 
   //...读取数据
   _rwlock.ReleaseReaderLock(); //释放读锁
   // 请求写锁，如果100ms超时退出 
   _rwlock.AcquireWriterLock(100); 
   //...写入数据
   _rwlock.ReleaseWriterLock(); //释放写锁
   ```

3. 系统级别的：信号量（Semaphore），互斥（Mutex），事件(AutoResetEvent/ManualResetEvent) 。线程池

   1. 信号量（Semaphore）：

      ```
       //当前允许同时访问线程数2; 最大允许数量=3 
       static Semaphore s = new Semaphore(2, 3); 
       static void Main(string[] args)
       {
           for (int i = 0; i < 10; i++)
               new Thread(Go).Start();
           Console.ReadLine();
       }
       static void Go()
       {
           while (true)
           {
               //WaitOne()和Release()必须成对出现，否则会一直等待/释放出错
               s.WaitOne();
               Thread.Sleep(1000); //因为当前数量为2，所以每sleep后输出2行 
               Console.WriteLine(Thread.CurrentThread.ManagedThreadId);
               s.Release();
           }
       }
      ```

      1. 互斥（Mutex）:

      ```
       var _mtx = new Mutex();
       var _resource = new List<string>();
       // 设置超时时限并在wait前退出非默认托管上下文 
       if (_mtx.WaitOne(1000, true))
       {
           _resource.Add("123");
           _mtx.ReleaseMutex();
           //WaitOne和ReleaseMutex必须成对出现，
           //否则会导致进程死锁的发生，会抛出AbandonedMutexException异常。
       }
      ```

      1. 事件(AutoResetEvent/ManualResetEvent)

      ```
       //处理线程等待的对象
       //false是手动set唤醒，如果传true则会在线程执行时自动set
       static EventWaitHandle wh = new AutoResetEvent(false);
       static void Main()
       {
           new Thread(Waiter).Start();//1
           new Thread(Waiter).Start();//2
           new Thread(Waiter).Start();//3
           Thread.Sleep(3000); //做任何耗时操作
           //唤醒，继续执行刚才第一个线程输出，然后再自动挂起
           wh.Set();  
           //再次唤醒会执行第二个线程输出，然后再自动挂起，AutoResetEvent是按顺序来的
           //而ManuelResetEvent则在执行Set()后释放了所有挂起的线程
           //wh.Set(); 
           Console.ReadKey();
       }
       static void Waiter()
       {
           Console.WriteLine("Waiting...");
           wh.WaitOne(); //等待唤醒之后继续往下执行
           //AutoResetEvent和ManuelResetEvent相比相当于自动执行了这行
           //mre.Reset(); 
           Console.WriteLine("Notified");
       }
      ```

      ManualResetEvent与AutoResetEvent区别是M的Set()方法一次释放所有挂起的线程，A在Set()之后又自动将后续线程挂起，需要再次Set()。A相当于M在WaitOne()之后执行M.Reset()

      1. 线程池（em.....暂时看不懂搜出来的东西）



## Q：Memochached（与redis区别）、UML用过吗？

A：用过Memochached，与redis的区别是redis有机制不是所有数据都放在内存中，会放磁盘一部分（重启不丢失）。redis支持更多数据类型（list,set,zset,hash）,redis是单线程。Memcached单个value最大只支持1MB，Redis最大支持512MB。UML没用过。

## Q：EF避免脏读？

A：我们只需要在实体中增加一个byte[]类型的字段，并为其加上[Timestamp]特性:

```
public class Entity
{
	public string Name {get; set;}
	[Timestamp]
	public byte[] v {get; set;}	
}
```

这样在更新或删除操作生成的Sql中，Where语句将包含 and v = @2 条件，如果有其它用户更改过此行，那么行版本将不一致，因此更新或删除sql会无法找到要更新的行，此时EF将认定该操作出现了并发冲突。 如果是控制某个列的并发，将ConcurrencyCheck特性添加到实体需要控制并发的**非主键属性**上即可。

## Q：说说你知道的数据库优化方式？

A：

1. 建立索引
2. 使用存储过程
3. 使用视图
4. 优化查询语句



## Q：数据库索引使用需要注意什么？什么是视图？如何做查询优化？

A：

1. 一个表内索引最好不要太多，一般在主键和经常做where条件或者group by或者order by的列上创建索引，如果表数据太小索引反而会影响性能。
2. 视图是由一张或多张表联合查询出的虚拟表，复杂查询使用视图来代替联表查询会提升性能。
3. 查询语句优化一般尽量避免全表扫描，以下几点会全表扫描：
   1. 使用null判断，会全表扫描。
   2. <>(不等于)判断，会全表扫描。
   3. 使用or，会全表扫描。
   4. 左右模糊查询%xx%，会全表扫描。
   5. 尽量使用exists代替in，in会全表扫描。
   6. 尽量不要在where语句中判断运算后的结果，会全表扫描。





## Q：什么是跨域？如何解决跨域问题？

A：跨域是为了限制JS和Cookie只能访问同域名下的资源。解决可以使用CORS（Cross-Orign Resouces Sharing跨域资源共享），nuget有现成的包，原理是在HTTP报文头中添加标识来控制。比如：Access-Control-Allow:[http://xx.com](http://xx.com/) 代表允许xx.com访问

## Q：Http管道模型是什么？HttpHandler是什么？HttpModule是什么？中间件是什么？

A：

1. Http管道模型是指一个请求一层一层达到action，然后再一层一层出去的过程像个管道一样。
2. HttpHandle是ISAPI的aspnet_isapi.dll通过后缀名然后转发到不同的处理程序。比如.aspx会调用System.Web.UI.PageHandler处理页面。
3. HttpModule是请求在管道中经过的模块，可以自行编写实现身份认证、过滤器等功能。
4. 中间件类似于HttpModule，也是在请求在管道中经过的模块，区别是HttpModule基于事件。



## Q：数组是否值类型？

A：数组都是引用类型，继承自System.Array，而Array继承自System.Object，众所周知值类型都是继承System.ValueType的。

## Q：a="123"; b="123";内存怎么分布，b=a时呢？

A：

1. “123”由于CLR的字符串驻留机制只存有一份。所以当a="123";b="123";时a和b都指向了"123"这个字符串的引用。
2. 当b=a时是把a的引用copy了一份给b。



## Q：const和readonly的区别？

A：const是编译时定好无法改变的。readonly归根结底还是对象中的属性，是动态的，只不过修饰为只读而已。



## Q：扩展方法是动态还是静态？怎么写扩展方法？

A：扩展方法是静态的，只能存在于静态类中。写法跟函数差不多，第一个参数必写为this 要扩展的类型 xxx，然后xxx则是调用扩展方法的对象本身，比如扩展一个string类的方法：

```
static void Output(this string str)
{
    Console.WriteLine(str);
    //"123".Output(); 执行后会输出123，
    //str对象就是调用方法的"123"字符串
}
```

## Q：DDD的理解和认识？什么是失血/贫血/充血/胀血模型？

A：可以通过聚合、实体等概念梗清晰的描述业务，底层的CQRS（读写分离）、UOW（UnitOfWork，自动实现事务）、仓储层都很好用。

1. 失血模型：领域模型只有get/set。没有任何实体业务逻辑，完全依赖service->DAL->domain调用。
2. 贫血模型：领域模型除了get/set后包含一些简单的实体组装逻辑，还是依赖service->DAL->domain调用。
3. 充血模型：领域模型包含DAL层的东西比如持久化，调用变为service->domain->DAL
4. 胀血模型：取消service层，直接通过domain->DAL来调用执行业务

**（血越多领域层越复杂，对service层依赖越小）**

## Q：如何全局处理异常？

A：使用中间件记录处理异常，或者使用ExceptionFilter来捕获全局异常。

## Q：nuget包上传管理流程？

A：

1. nuget官网下载nuget.exe
2. cmd命令：nuget pack生成x.nupkg文件
3. 登录nuget官网创建一个密钥
4. cmd命令：nuget push x.nupkg 密钥 -Source https://api.nuget.org/v3/index.json
5. 上传完成，可在VS中搜索到



## Q：泛型的内部机制？

A：以List为例，第一次编译时只是为T生成一个占位符。在实际用到时比如List时，JIT(Just-In-Time即时编译器)会用User去代替T的占位符实例化User，以此来实现泛型。



# 值类型和引用类型

#### 1，**值类型和引用类型的区别？**

值类型包括简单类型、结构体类型和枚举类型，引用类型包括自定义类、数组、接口、委托等。

- 1、赋值方式：将一个值类型变量赋给另一个值类型变量时，将复制包含的值。这与引用类型变量的赋值不同，引用类型变量的赋值只复制对象的引用（即内存地址，类似C++中的指针），而不复制对象本身。
- 2、继承：值类型不可能派生出新的类型，所有的值类型均隐式派生自 System.ValueType。但与引用类型相同的是，结构也可以实现接口。
- 3、null：与引用类型不同，值类型不可能包含 null 值。然而，可空类型允许将 null 赋给值类型（他其实只是一种语法形式，在clr底层做了特殊处理）。
- 4、每种值类型均有一个隐式的默认构造函数来初始化该类型的默认值，值类型初始会默认为0，引用类型默认为null。
- 5、值类型存储在栈中，引用类型存储在托管堆中。



#### 2，**结构和类的区别？**

结构体是值类型，类是引用类型，主要区别如题1。其他的区别：

- 结构不支持无惨构造函数，不支持析构函数，并且不能有protected修饰；
- 结构常用于数据存储，类class多用于行为；
- class需要用new关键字实例化对象，struct可以不适用new关键字；
- class可以为抽象类，struct不支持抽象；

#### 3，**delegate是引用类型还是值类型？enum、int[]和string呢？**

enum枚举是值类型，其他都是引用类型。

#### 4，**堆和栈的区别？**

```
线程堆栈：简称栈 Stack
托管堆： 简称堆 Heap
```

- 值类型大多分配在栈上，引用类型都分配在堆上；
- 栈由操作系统管理，栈上的变量在其作用域完成后就被释放，效率较高，但空间有限。堆受CLR的GC控制；
- 栈是基于线程的，每个线程都有自己的线程栈，初始大小为1M。堆是基于进程的，一个进程分配一个堆，堆的大小由GC根据运行情况动态控制；



#### 5，**什么情况下会在堆（栈）上分配数据？它们有性能上的区别吗？**



#### 6，“结构”对象可能分配在堆上吗？什么情况下会发生，有什么需要注意的吗？

结构是值类型，有两种情况会分配在对上面：

- 结构作为class的一个字段或属性，会随class一起分配在堆上面；
- 装箱后会在堆中存储，尽量避免值类型的装箱，值类型的拆箱和装箱都有性能损失



#### 7，理解参数按值传递？以及按引用传递？

- 按值传递：对于值类型传递的它的值拷贝副本，而引用类型传递的是引用变量的内存地址，他们还是指向的同一个对象。
- 按引用传递：通过关键字out和ref传递参数的内存地址，值类型和引用类型的效果是相同的。



#### 8，**`out` 和 `ref` 的区别与相同点？**

- `out` 和 `ref`都指示编译器传递参数地址，在行为上是相同的；
- 他们的使用机制稍有不同，ref要求参数在使用之前要显式初始化，out要在方法内部初始化；
- `out` 和 `ref`不可以重载，就是不能定义Method(ref int a)和Method(out int a)这样的重载，从编译角度看，二者的实质是相同的，只是使用时有区别；



#### 9，**C#支持哪几个预定义的值类型？C#支持哪些预定义的引用类型？**

值类型：整数、浮点数、字符、bool和decimal

引用类型：Object，String



#### 10，**有几种方法可以判定值类型和引用类型？**

简单来说，继承自System.ValueType的是值类型，反之是引用类型。



#### 11，**说说值类型和引用类型的生命周期？**

值类型在作用域结束后释放。

引用类型由GC垃圾回收期回收

#### 12，**如果结构体中定义引用类型，对象在内存中是如何存储的？例如下面结构体中的class类 User对象是存储在栈上，还是堆上？**

```
public struct MyStruct 
{ 
    public int Index; 
    public User User; 
```

![img](https://images.cnblogs.com/cnblogs_com/anytao/typesystem.gif)



![img](https://pic002.cnblogs.com/img/fan0136/200902/2009020510331710.jpg)



#### 13，对象拷贝

```c#
			int v1 = 0;
            int v2 = v1;
            v2 = 100;
            Console.WriteLine("v1=" + v1); //输出：v1=0
            Console.WriteLine("v2=" + v2); //输出：v2=100

            User u1=new User();
            u1.Age = 0;
            User u2 = u1;
            u2.Age = 100;
            Console.WriteLine("u1.Age=" + u1.Age); //输出：u1.Age=100
            Console.WriteLine("u2.Age=" + u2.Age); //输出：u2.Age=100，因为u1/u2指向同一个对象
```

#### 14，out和ref的关系

```c#
		private void DoTest( ref int a)
        {
            a *= 2;
        }

        private void DoUserTest(ref User user)
        {
            user.Age *= 2;
        }

        [NUnit.Framework.Test]
        public void DoParaTest()
        {
            int a = 10;
            DoTest(ref a);
            Console.WriteLine("a=" + a); //输出：a=20 ,a的值改变了
            User user = new User();
            user.Age = 10;
            DoUserTest(ref user);
            Console.WriteLine("user.Age=" + user.Age); //输出：user.Age=20
        }
```

**`out` 和 `ref`的主要异同：**

- `out` 和 `ref`都指示编译器传递参数地址，在行为上是相同的；
- 他们的使用机制稍有不同，ref要求参数在使用之前要显式初始化，out要在方法内部初始化；
- `out` 和 `ref`不可以重载，就是不能定义Method(ref int a)和Method(out int a)这样的重载，从编译角度看，二者的实质是相同的，只是使用时有区别；

#### 15，常见问题

![img](https://images.cnblogs.com/cnblogs_com/gaoyuchuanit/%D6%B5%C0%E0%D0%CD%BA%CD%D2%FD%D3%C3%C0%E0%D0%CD%C7%F8%B1%F0.jpg)

**Refences**

https://www.cnblogs.com/anding/p/5226343.html  .net面试题解析

# 拆箱和装箱

#### 1，什么是拆箱和装箱？

装箱就是值类型转换为引用类型，拆箱就是引用类型（被装箱的对象）转换为值类型。

#### 2，什么是箱子？

就是引用类型对象

#### 3，箱子放在哪里？

托管堆上。

#### 4，装箱和拆箱有什么性能影响？

装箱和拆箱都涉及到内存的分配和对象的创建，有较大的性能影响。

#### 5，如何避免隐身装箱？

编码中，多使用泛型、显示装箱。

#### 6，箱子的基本结构？

箱子就是一个引用类型对象，因此她的结构，主要包含两部分：

- 值类型字段值；
- 引用类型的标准配置，引用对象的额外空间：TypeHandle和同步索引块

#### 7，装箱的过程？

- 1.在堆中申请内存，内存大小为值类型的大小，再加上额外固定空间（引用类型的标配：TypeHandle和同步索引块）；
- 2.将值类型的字段值（x=1023）拷贝新分配的内存中；
- 3.返回新引用对象的地址（给引用变量object o）

#### 8，拆箱的过程？

- 1.检查实例对象（object o）是否有效，如是否为null，其装箱的类型与拆箱的类型（int）是否一致，如检测不合法，抛出异常；
- 2.指针返回，就是获取装箱对象（object o）中值类型字段值的地址；
- 3.字段拷贝，把装箱对象（object o）中值类型字段值拷贝到栈上，意思就是创建一个新的值类型变量来存储拆箱后的值；

#### 9，下面这段代码输出什么？共发生多少次装箱？多少次拆箱？

```
int i = 5;
object obj = i;
IFormattable ftt = i;
Console.WriteLine(System.Object.ReferenceEquals(i, obj));
Console.WriteLine(System.Object.ReferenceEquals(i, ftt));
Console.WriteLine(System.Object.ReferenceEquals(ftt, obj));
Console.WriteLine(System.Object.ReferenceEquals(i, (int)obj));
Console.WriteLine(System.Object.ReferenceEquals(i, (int)ftt));
```

所有值类型都是继承自System.ValueType，而System.ValueType又是来自何方呢，不难发现System.ValueType继承自System.Object。因此**Object是.NET中的万物之源**，几乎所有类型都来自她，这是装箱与拆箱的基础。

#### 10，基本概念：

拆箱与装箱就是值类型与引用类型的转换，她是值类型和引用类型之间的桥梁，他们可以相互转换的一个基本前提就是上面所说的：Object是.NET中的万物之源

```
      int x = 1023;
            object o = x; //装箱
            int y = (int) o; //拆箱
```

**装箱**：值类型转换为引用对象，一般是转换为System.Object类型或值类型实现的接口引用类型；

**拆箱**：引用类型转换为值类型，注意，这里的引用类型只能是被装箱的引用类型对象；

#### 11，装箱过程

装箱就是把值类型转换为引用类型，具体过程：

- 1.在堆中申请内存，内存大小为值类型的大小，再加上额外固定空间（引用类型的标配：TypeHandle和同步索引块）；
- 2.将值类型的字段值（x=1023）拷贝新分配的内存中；
- 3.返回新引用对象的地址（给引用变量object o）

装箱后内存有两个对象：一个是值类型变量x，另一个就是新引用对象o。装箱对应的IL指令为`box`，上面装箱的IL代码如下图：

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160302221928611-1465400260.png)

#### 12，拆箱过程

明白了装箱，拆箱就是装箱相反的过程，简单的说是把装箱后的引用类型转换为值类型。具体过程：

- 1.检查实例对象（object o）是否有效，如是否为null，其装箱的类型与拆箱的类型（int）是否一致，如检测不合法，抛出异常；
- 2.指针返回，就是获取装箱对象（object o）中值类型字段值的地址；
- 3.字段拷贝，把装箱对象（object o）中值类型字段值拷贝到栈上，意思就是创建一个新的值类型变量来存储拆箱后的值；

如上图所示，拆箱后，得到一个新的值类型变量y，拆箱对应的IL指令为`unbox`，拆箱的IL代码如下：

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160302221930845-450120673.png)

通过上面深入了解了装箱与拆箱的原理，不难理解，只有值类型可以装箱，拆的就是装箱后的引用对象，箱子就是一个存放了值类型字段的引用对象实例，箱子存储在托管堆上。**只有值类型才有装箱、拆箱两个状态，而引用类型一直都在箱子里**。

#### 13，性能问题

之所以关注装箱与拆箱，主要原因就是他们的性能问题，而且在日常编码中，经常有装箱与拆箱的操作，而且这些装箱与拆箱的操作往往是在不经意时发生。**一般来说，装箱的性能开销更大**，这不难理解，因为引用对象的分配更加复杂，成本也更高，值类型分配在栈上，分配和释放的效率都很高。装箱过程是需要创建一个新的引用类型对象实例，拆箱过程需要创建一个值类型字段，开销更低。

为了尽量避免这种性能损失，尽量使用泛型，在代码编写中也尽量避免隐式装箱。

#### 14，隐式装箱如何避免

就是不经意的代码导致多次重复的装箱操作，看看代码就好理解了

```
int x = 100;
ArrayList arr = new ArrayList(3);
arr.Add(x);
arr.Add(x);
arr.Add(x);
```

这段代码共有多少次装箱呢？看看Add方法的定义：

[![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160302221931611-1608957151.png)](http://images2015.cnblogs.com/blog/151257/201603/151257-20160302221931267-804400960.png)

再看看IL代码，可以准确的得到装箱的次数：

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160302221932861-570048436.png)

显示装箱可以避免隐式装箱，下面修改后的代码就只有一次装箱了。

```c#
int x = 100;
ArrayList arr = new ArrayList(3);
object o = x;
arr.Add(o);
arr.Add(o);
arr.Add(o);
```

# string与字符串操作

#### 1，字符串是引用类型类型还是值类型？

引用类型。

#### 2，在字符串连接处理中，最好采用什么方式，理由是什么？

少量字符串连接，使用String.Concat，大量字符串使用StringBuilder，因为StringBuilder的性能更好，如果string的话会创建大量字符串对象。

#### 3，使用 StringBuilder时，需要注意些什么问题？

- 少量字符串时，尽量不要用，StringBuilder本身是有一定性能开销的；
- 大量字符串连接使用StringBuilder时，应该设置一个合适的容量；

#### 4，以下代码执行后内存中会存在多少个字符串？分别是什么？输出结果是什么？为什么呢？

```c#
string st1 = "123" + "abc";
string st2 = "123abc";
Console.WriteLine(st1 == st2); //True
Console.WriteLine(System.Object.ReferenceEquals(st1, st2));//True
```

#### 5，以下代码执行后内存中会存在多少个字符串？分别是什么？输出结果是什么？为什么呢？

```c#
string s1 = "123";
string s2 = s1 + "abc";
string s3 = "123abc";
Console.WriteLine(s2 == s3);
Console.WriteLine(System.Object.ReferenceEquals(s2, s3));
```

内存中的字符串只有一个“123abc”,第一行代码（string st1 = "123" + "abc"; ）常量字符串相加会被编译器优化。由于字符串驻留机制，两个变量st1、st2都指向同一个对象。

#### 6，使用C#实现字符串反转算法，例如：输入"12345", 输出"54321"。

#### 7，下面的代码输出结果？为什么？

```c#
object a = "123";
object b = "123";
Console.WriteLine(System.Object.Equals(a,b));
Console.WriteLine(System.Object.ReferenceEquals(a,b));
string sa = "123";
Console.WriteLine(System.Object.Equals(a, sa));
Console.WriteLine(System.Object.ReferenceEquals(a, sa));
```

输出结果全是True，因为他们都指向同一个字符串实例，使用object声明和string声明在这里并没有区别（string是引用类型）。

#### 8，认识string

首先需要明确的，string是一个引用类型，其对象值存储在托管堆中。string的内部是一个char集合，他的长度Length就是字符char数组的字符个数。string不允许使用new string()的方式创建实例，而是另一种更简单的语法，直接赋值（string aa= “000”这一点也类似值类型）。

认识string，先从一个简单的示例代码入手：

```
public void DoStringTest()
{
    var aa = "000";
    SetStringValue(aa);
    Console.WriteLine(aa);
}

private void SetStringValue(string aa)
{
    aa += "111";
}
```

```
上面的输出结果为“000”。
```

通过前面的值类型与引用类型的文章，我们知道string是一个引用类型，既然是一个引用类型，参数传递的是引用地址，那为什么不是输出“000111”呢？是不是很有值类型的特点呢！这一切的原因源于string类型的两个重要的特性：**恒定性**与**驻留性**

#### 9，string的恒定性

字符串是不可变的，字符串一经创建，就不会改变，任何改变都会产生新的字符串。比如下面的代码，堆上先创建了字符串s1=”a”，加上一个字符串“b”后，堆上会存在三个个字符串实例，如下图所示。

```c#
string s1 = "a";
string s2 = s1 + "b";
```

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160303222417112-1147871973.png)

文中的”**任何改变都会产生新的字符串**“，包括字符串的一些操作函数，如str1.ToLower，Trim()，Remove(int startIndex, int count)，ToUpper()等，都会产生新的字符串，因此在很多编程实践中，对于字符串忽略大小的比较：

```c#
if（str1.ToLower()==str2.ToLower()） //这种方式会产生新的字符串，不推荐
if（string. Compare(str1,str2,true)） //这种方式性能更好
```

#### 10，string的驻留性

由于字符串的不变性，在大量使用字符串操作时，会导致创建大量的字符串对象，带来极大的性能损失。因此CLR又给string提供另外一个法宝，就是字符串驻留，先看看下面的代码，字符串s1、s2竟然是同一个对象！

```
var s1 = "123";
var s2 = "123";
Console.WriteLine(System.Object.Equals(s1, s2));  //输出 True
Console.WriteLine(System.Object.ReferenceEquals(s1, s2));  //输出 True
```

相同的字符串在内存（堆）中只分配一次，第二次申请字符串时，发现已经有该字符串是，直接返回已有字符串的地址，这就是驻留的基本过程。

**字符串驻留的基本原理：**

- CLR初始化时会在内存中创建一个驻留池，内部其实是一个哈希表，存储被驻留的字符串和其内存地址。
- 驻留池是进程级别的，多个AppDomain共享。同时她不受GC控制，生命周期随进程，意思就是不会被GC回收（不回收！难道不会造成内存爆炸吗？不要急，且看下文）
- 当分配字符串时，首先会到驻留池中查找，如找到，则返回已有相同字符串的地址，不会创建新字符串对象。如果没有找到，则创建新的字符串，并把字符串添加到驻留池中。

如果大量的字符串都驻留到内存里，而得不到释放，不是很容易造成内存爆炸吗，当然不会了？因为不是任何字符串都会驻留，只有通过IL指令`ldstr`创建的字符串才会留用。

字符串创建的有多种方式，如下面的代码：

```c#
var s1 = "123";
var s2 = s1 + "abc";
var s3 = string.Concat(s1, s2);
var s4 = 123.ToString();
var s5 = s2.ToUpper();
```

其IL代码如下

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160303221217190-205612505.png)

**在上面的代码中，出现两个字符串常量，“123”和“abc”，这个两个常量字符串在IL代码中都是通过IL指令`ldstr`创建的，只有该指令创建的字符串才会被驻留，其他方式产生新的字符串都不会被驻留，也就不会共享字符串了，会被GC正常回收**。

那该如何来验证字符串是否驻留呢，string类提供两个静态方法：

- String.Intern(string str) 可以主动驻留一个字符串；
- String.IsInterned(string str);检测指定字符串是否驻留，如果驻留则返回字符串，否则返回NULL

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160303221218221-484836220.png)

请看下面的示例代码

```c#
var s1 = "123";
var s2 = s1 + "abc";
Console.WriteLine(s2);   //输出：123abc
Console.WriteLine(string.IsInterned(s2) ?? "NULL");   //输出：NULL。因为“123abc”没有驻留

string.Intern(s2);   //主动驻留字符串
Console.WriteLine(string.IsInterned(s2) ?? "NULL");   //输出：123abc
```

#### 11，StringBuilder

大量的编程实践和意见中，都说大量字符串连接操作，应该使用StringBuilder。相对于string的不可变，StringBuilder代表可变字符串，不会像字符串，在托管堆上频繁分配新对象，StringBuilder是个好同志。

首先StringBuilder内部同string一样，有一个**char[]字符数组**，负责维护字符串内容。因此，与char数组相关，就有两个很重要的属性：

- public int **Capacity**：StringBuilder的容量，其实就是字符数组的长度。
- public int **Length**：StringBuilder中实际字符的长度，>=0，<=容量Capacity。

StringBuilder之所以比string效率高，主要原因就是不会创建大量的新对象，StringBuilder在以下两种情况下会分配新对象：

- **追加字符串时，当字符总长度超过了当前设置的容量Capacity，这个时候，会重新创建一个更大的字符数组，此时会涉及到分配新对象。**
- **调用StringBuilder.ToString()，创建新的字符串。**

**追加字符串的过程：**

- StringBuilder的默认初始容量为16；
- 使用stringBuilder.Append()追加一个字符串时，当字符数大于16，StringBuilder会自动申请一个更大的字符数组，一般是倍增；
- 在新的字符数组分配完成后，将原字符数组中的字符复制到新字符数组中，原字符数组就被无情的抛弃了（会被GC回收）；
- 最后把需要追加的字符串追加到新字符数组中；

简单来说，当StringBuilder的容量**Capacity**发生变化时，就会引起托管对象申请、内存复制等操作，带来不好的性能影响，因此设置合适的初始容量是非常必要的，尽量减少内存申请和对象创建。代码简单来验证一下：

```c#
StringBuilder sb1 = new StringBuilder();
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=16; Length=0;   //初始容量为16 
sb1.Append('a', 12);    //追加12个字符
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=16; Length=12;  
sb1.Append('a', 20);    //继续追加20个字符，容量倍增了
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=32; Length=32;  
sb1.Append('a', 41);    //追加41个字符，新容量=32+41=73
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=73; Length=73;  

StringBuilder sb2 = new StringBuilder(80); //设置一个合适的初始容量
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=0;
sb2.Append('a', 12);
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=12;
sb2.Append('a', 20);
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=32;
sb2.Append('a', 41);
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=73;
```

为什么少量字符串不推荐使用StringBuilder呢？因为StringBuilder本身是有一定的开销的，少量字符串就不推荐使用了，使用String.Concat和String.Join更合适。

- 在使用线程锁的时候，不要锁定一个字符串对象，因为字符串的驻留性，可能会引发不可以预料的问题；
- 理解字符串的不变性，尽量避免产生额外字符串，如：

```
if（str1.ToLower()==str2.ToLower()） //这种方式会产生新的字符串，不推荐
if（string. Compare(str1,str2,true)） //这种方式性能更好
```

- 在处理大量字符串连接的时候，尽量使用StringBuilder，在使用StringBuilder时，尽量设置一个合适的长度初始值；
- 少量字符串连接建议使用String.Concat和String.Join代替。



# 类型，方法和继承

#### 1，所有类型都继承System.Object吗？

基本上是的，所有值类型和引用类型都继承自System.Object，接口是一个特殊的类型，不继承自System.Object。

#### 2，解释virtual、sealed、override和abstract的区别

- virtual申明虚方法的关键字，说明该方法可以被重写
- sealed说明该类不可被继承
- override重写基类的方法
- abstract申明抽象类和抽象方法的关键字，抽象方法不提供实现，由子类实现，抽象类不可实例化。



#### 3，接口和类有什么异同？

**不同点：**

1、接口不能直接实例化。

2、接口只包含方法或属性的**声明**，不包含方法的实现。

3、接口可以**多继承**，类只能单继承。

5、表达的含义不同，接口主要定义一种**规范**，统一调用方法，也就是规范类，约束类，类是方法功能的实现和集合

**相同点：**

1、接口、类和结构都可以从多个接口继承。

2、接口类似于抽象基类：继承接口的任何非抽象类型都必须实现接口的所有成员。

3、接口和类都可以包含事件、索引器、方法和属性。

#### 4，抽象类和接口有什么区别？使用时有什么需要注意的吗？

1、继承：接口支持多继承；抽象类不能实现多继承。

2、表达的概念：接口用于规范，更强调契约，抽象类用于共性，强调父子。抽象类是一类事物的高度聚合，那么对于继承抽象类的子类来说，对于抽象类来说，属于"Is A"的关系；而接口是定义行为规范，强调“Can Do”的关系，因此对于实现接口的子类来说，相对于接口来说，是"行为需要按照接口来完成"。

3、方法实现：对抽象类中的方法，即可以给出实现部分，也可以不给出；而接口的方法（抽象规则）都不能给出实现部分，接口中方法不能加修饰符。

4、子类重写：继承类对于两者所涉及方法的实现是不同的。继承类对于抽象类所定义的抽象方法，可以不用重写，也就是说，可以延用抽象类的方法；而对于接口类所定义的方法或者属性来说，在继承类中必须重写，给出相应的方法和属性实现。

5、新增方法的影响：在抽象类中，新增一个方法的话，继承类中可以不用作任何处理；而对于接口来说，则需要修改继承类，提供新定义的方法。

6、接口可以作用于值类型（枚举可以实现接口）和引用类型；抽象类只能作用于引用类型。

7、接口不能包含字段和已实现的方法，接口只包含方法、属性、索引器、事件的签名；抽象类可以定义字段、属性、包含有实现的方法。

#### 5， 重载与覆盖的区别？

重载：当类包含两个名称相同但签名不同(方法名相同,参数列表不相同)的方法时发生方法重载。用方法重载来提供在语义上完成相同而功能不同的方法。

覆写：在类的继承中使用，通过覆写子类方法可以改变父类虚方法的实现。

**主要区别**：

1、方法的覆盖是子类和父类之间的关系，是垂直关系；方法的重载是同一个类中方法之间的关系，是水平关系。
2、覆盖只能由一个方法，或只能由一对方法产生关系；方法的重载是多个方法之间的关系。
3、覆盖要求参数列表相同；重载要求参数列表不同。
4、覆盖关系中，调用那个方法体，是根据对象的类型来决定；重载关系，是根据调用时的实参表与形参表来选择方法体的。

#### 6，在继承中new和override相同点和区别？看下面的代码，有一个基类A，B1和B2都继承自A，并且使用不同的方式改变了父类方法Print（）的行为。测试代码输出什么？为什么？

```c#
public void DoTest()
{
    B1 b1 = new B1(); B2 b2 = new B2();
    b1.Print(); b2.Print();      //按预期应该输出 B1、B2

    A ab1 = new B1(); A ab2 = new B2();
    ab1.Print(); ab2.Print();   //这里应该输出什么呢？
}
public class A
{
    public virtual void Print() { Console.WriteLine("A"); }
}
public class B1 : A
{
    public override void Print() { Console.WriteLine("B1"); }
}
public class B2 : A
{
    public new void Print() { Console.WriteLine("B2"); }
}
```

#### 7，下面代码中，变量a、b都是int类型，代码输出结果是什么？

```c#
int a = 123;
int b = 20;
var atype = a.GetType();
var btype = b.GetType();
Console.WriteLine(System.Object.Equals(atype,btype));
Console.WriteLine(System.Object.ReferenceEquals(atype,btype));
```

#### 8，class中定义的静态字段是存储在内存中的哪个地方？为什么会说她不会被GC回收？

类型对象存储在内存的加载堆上，因为加载堆不受GC管理，其生命周期随AppDomain，不会被GC回收。

#### 9，类型Type

![img](https://images.cnblogs.com/cnblogs_com/anytao/Anytao_15_3.jpg)

![image](https://images.cnblogs.com/cnblogs_com/artech/WindowsLiveWriter/CLR_12360/image_thumb_1.png)



![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160306225937768-965773594.png)



**方法表的加载**：

- 方法表的加载时父类在前子类在后的，首先加载的是固定的4个来自System.Object的虚方法：ToString, Equals, GetHashCode, and Finalize；
- 然后加载父类A的虚方法；
- 加载自己的方法；
- 最后是构造方法：静态构造函数.cctor()，对象构造函数.ctor()；

**方法的调用**：当执行代码`b1.Print()`时（此处只关注方法调用，忽略方法的继承等因素），通过b1的TypeHandel找到对应类型对象，然后找到方法表槽，然后是对应的IL代码，第一次执行的时候，JIT编译器需要把IL代码编译为本地机器码，第一次执行完成后机器码会保留，下一次执行就不需要JIT编译了。这也是为什么说.NET程序启动需要预热的原因。

#### 10，继承本质

方法表的创建过程是从父类到子类自上而下的，这是.NET中继承的很好体现，**当发现有覆写父类虚方法会覆盖同名的父方法**，所有类型的加载都会递归到System.Object类。

- 继承是可传递的，子类是对父类的扩展，必须继承父类方法，同时可以添加新方法。
- 子类可以调用父类方法和字段，而父类不能调用子类方法和字段。 
- 子类不光继承父类的公有成员，也继承了私有成员，只是不可直接访问。
- new关键字在虚方法继承中的阻断作用，中断某一虚方法的继承传递。

因此类型B1、B2的类型对象进一步的结构示意图如下：

- 在加载B1类型对象时，当加载override B1.Print(“B1”)时，发现有覆写override的方法，会覆盖父类的同名虚方法Print(“A”)，就是下面的示意图，简单来说就是在B1中Print只有一个实现版本；
- 加载B2类型对象时，new关键字表示要隐藏基类的虚方法，此时B2中的Print(“B2”)就不是虚方法了，她是B2中的新方法了，简单来说就是在B2类型对象中Print有2个实现版本；

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160306225939455-388265033.png)



**抽象类提供多个派生类共享基类的公共定义，它既可以提供抽象方法，也可以提供非抽象方法。抽象类不能实例化，必须通过继承由派生类实现其抽象方法，因此对抽象类不能使用new关键字，也不能被密封。**

基本特点：

- 抽象类使用Abstract声明，抽象方法也是用Abstract标示；
- 抽象类不能被实例化；
- 抽象方法必须定义在抽象类中；
- 抽象类可以继承一个抽象类；
- 抽象类不能被密封（不能使用sealed）；
- 同类Class一样，只支持单继承；

#### 11，接口

基本特点：

- 接口使用interface声明；
- 接口类似于抽象基类，不能直接实例化接口；
- 接口中的方法都是抽象方法，不能有实现代码，实现接口的任何非抽象类型都必须实现接口的所有成员：
- 接口成员是自动公开的，且不能包含任何访问修饰符。
- 接口自身可从多个接口继承，类和结构可继承多个接口，但接口不能继承类。

关于继承，太概念性了，就不细说了，主要还是在平时的搬砖过程中多思考、多总结、多体会。在.NET中继承的主要两种方式就是**类继承和接口继承**，两者的主要思想是不一样的：

- 类继承强调父子关系，是一个“IS A”的关系，因此只能单继承（就像一个人只能有一个Father）；
- 接口继承强调的是一种规范、约束，是一个“CAN DO”的关系，支持多继承，是实现多态一种重要方式。



# 常量、字段、属性、特性与委托

#### 1，const和readonly有什么区别？

const关键字用来声明编译时常量，readonly用来声明运行时常量。都可以标识一个常量，主要有以下区别：
1、初始化位置不同。const必须在声明的同时赋值；readonly即可以在声明处赋值，也可以在构造方法里赋值。
2、修饰对象不同。const即可以修饰类的字段，也可以修饰局部变量；readonly只能修饰类的字段 。
3、const是编译时常量，在编译时确定该值，且值在编译时被内联到代码中；readonly是运行时常量，在运行时确定该值。
4、const默认是静态的；而readonly如果设置成静态需要显示声明 。
5、支持的类型时不同，const只能修饰基元类型或值为null的其他引用类型；readonly可以是任何类型。

#### 2，哪些类型可以定义为常量？常量const有什么风险？

基元类型或值为null的其他引用类型，常量的风险就是不支持跨程序集版本更新，常量值更新后，所有使用该常量的代码都必须重新编译。

#### 3，字段与属性有什么异同？

- 属性提供了更为强大的，灵活的功能来操作字段
- 出于面向对象的封装性，字段一般不设计为Public
- 属性允许在set和get中编写代码
- 属性允许控制set和get的可访问性，从而提供只读或者可读写的功能 （逻辑上只写是没有意义的）
- 属性可以使用override 和 new

#### 4， 静态成员和非静态成员的区别？

- 静态变量使用 static 修饰符进行声明，静态成员在加类的时候就被加载（上一篇中提到过，静态字段是随类型对象存放在Load Heap上的），通过类进行访问。
- 不带有static 修饰符声明的变量称做非静态变量，在对象被实例化时创建，通过对象进行访问 。
- 一个类的所有实例的同一静态变量都是同一个值，同一个类的不同实例的同一非静态变量可以是不同的值 。
- 静态函数的实现里不能使用非静态成员，如非静态变量、非静态函数等。

#### 5，自动属性有什么风险？

因为自动属性的私有字段是由编译器命名的，后期不宜随意修改，比如在序列化中会导致字段值丢失。

#### 6，特性是什么？如何使用？

特性与属性是完全不相同的两个概念，只是在名称上比较相近。Attribute特性就是关联了一个目标对象的一段配置信息，本质上是一个类，其为目标元素提供关联附加信息，这段附加信息存储在dll内的元数据，它本身没什么意义。运行期以反射的方式来获取附加信息。

使用方式参考https://www.cnblogs.com/anding/p/5129178.html

#### 7，下面的代码输出什么结果？为什么？

```c#
List<Action> acs = new List<Action>(5);
for (int i = 0; i < 5; i++)
{
    acs.Add(() => { Console.WriteLine(i); });
}
acs.ForEach(ac => ac());
```

输出了 5 5 5 5 5，全是5！因为闭包中的共享变量i会被提升为委托对象的公共字段，生命周期延长了

#### 8，C#中的委托是什么？事件是不是一种委托？

什么是委托？简单来说，委托类似于 C或 C++中的函数指针，允许将方法作为参数进行传递。

- C#中的委托都继承自System.Delegate类型；
- 委托类型的声明与方法签名类似，有返回值和参数；
- 委托是一种可以封装命名（或匿名）方法的引用类型，把方法当做指针传递，但委托是面向对象、类型安全的；

事件可以理解为一种特殊的委托，事件内部是基于委托来实现的。

#### 9，常量

常量的基本概念就不细说了，关于常量的几个特点总结一下：

- 常量的值必须在编译时确定，简单说就是在定义是设置值，以后都不会被改变了，她是编译常量。
- 常量只能用于简单的类型，因为常量值是要被编译然后保存到程序集的元数据中，只支持基元类型，如int、char、string、bool、double等。
- 常量在使用时，是把常量的值内联到IL代码中的，常量类似一个占位符，在编译时被替换掉了。正是这个特点导致常量的一个风险，就是**不支持跨程序集版本更新**；

关于常量**不支持跨程序集版本更新**，举个简单的例子来说明：

```
public class A
{
    public const int PORT = 10086;

    public virtual void Print()
    {
        Console.WriteLine(A.PORT);
    }
}
```

上面一段非常简单代码，其生产的IL代码如下，在使用常量变量的地方，把她的值拷过来了（把常量的值内联到使用的地方），与常量变量A.PORT没有关系了。假如A引用了B程序集（B.dll文件）中的一个常量，如果后面单独修改B程序集中的常量值，只是重新编译了B，而没有编译程序集A，就会出问题了，就是上面所说的不支持跨程序集版本更新。常量值更新后，所有使用该常量的代码都必须重新编译，这是我们在使用常量时必须要注意的一个问题。

- 不要随意使用常量，特别是有可能变化的数据；
- 不要随便修改已定义好的常量值；

接着上面的const说，其实枚举enum也有类似的问题，其根源和const一样，看看代码你就明白了。下面的是一个简单的枚举定义，她的IL代码定义和const定义是一样一样的啊！枚举的成员定义和常量定义一样，因此枚举其实本质上就相当是一个常量集合。

```
public enum EnumType : int
{
    None=0,
    Int=1,
    String=2,
}
```

#### 10，字段

字段本身没什么好说的，这里说一个字段的**内联初始化**问题吧，可能容易被忽视的一个小问题（不过好像也没什么影响），先看看一个简单的例子：

```
public class SomeType
{
    private int Age = 0;
    private DateTime StartTime = DateTime.Now;
    private string Name = "三体";
}
```

定义字段并初始化值，是一种很常见的代码编写习惯。但注意了，看看IL代码结构，一行代码（定义字段+赋值）被拆成了两块，最终的赋值都在构造函数里执行的。

那么问题来了，如果有多个构造函数，就像下面这样，**有多半个构造函数，会造成在两个构造函数.ctor中重复产生对字段赋值的IL代码，这就造成了不必要的代码膨胀**。这个其实也很好解决，在非默认构造函数后加一个“:this()”就OK了，或者显示的在构造函数里初始化字段。

#### 11，属性的本质

属性是面向对象编程的基本概念，提供了对私有字段的访问封装，在C#中以get和set访问器方法实现对可读可写属性的操作，提供了安全和灵活的数据访问封装。我们看看属性的本质，主要手段还是IL代码：

```
public class SomeType
{
    public int Index { get; set; }

    public SomeType() { }
}
```

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160308203232913-500597794.png)

上面定义的属性Index被分成了三个部分：

- 自动生成的私有字段“<Index>k__BackingField”
- 方法：get_Index()，获取字段值；
- 方法：set_Index(int32 'value')，设置字段值；

因此可以说属性的本质还是方法，使用面向对象的思想把字段封装了一下。在定义属性时，我们可以自定义一个私有字段，也可以使用**自动属性**“{ get; set; } ”的简化语法形式。

使用自动属性时需要注意一点的是，私有字段是由编译器自动命名的，是不受开发人员控制的。

#### 12，委托和事件

.NET中没有函数指针，方法也不可能传递，委托之所可以像一个普通引用类型一样传递，那是因为她本质上就是一个类。下面代码是一个非常简单的自定义委托：

```c#
public delegate void ShowMessageHandler(string mes);
```

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160308203234335-1390533361.png)

我们一行定义一个委托的代码，编译器自动生成了一堆代码：

- 编译器自动帮我们创建了一个类ShowMessageHandler，继承自System.MulticastDelegate（她又继承自System.Delegate），这是一个多播委托；
- 委托类ShowMessageHandler中包含几个方法，其中最重要的就是Invoke方法，签名和定义的方法签名一致；
- 其他两个版本BeginInvoke和EndInvoke是异步执行版本；

因此，也就不难猜测，当我们调用委托的时候，其实就是调用委托对象的Invoke方法，可以验证一下，下面的调用代码会被编译为对委托对象的Invoke方法调用：

```c#
private ShowMessageHandler ShowMessage;

//调用
this.ShowMessage("123");
```

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160308203235319-1179909381.png)

#### 13，闭包

闭包提供了一种类似脚本语言函数式编程的便捷、可以共享数据，但也存在一些隐患。

题目列表中的第7题，就是一个.NET的闭包的问题。

```
List<Action> acs = new List<Action>(5);
for (int i = 0; i < 5; i++)
{
    acs.Add(() => { Console.WriteLine(i); });
}
acs.ForEach(ac => ac()); // 输出了 5 5 5 5 5，全是5？这一定不是你想要的吧！这是为什么呢？
```

上面的代码中的Action就是.NET为我们定义好的一个无参数无返回值的委托，从上一节我们知道委托实质是一个类，理解这一点是解决本题的关键。在这个地方委托方法共享使用了一个局部变量i，那生成的类会是什么样的呢？看看IL代码：

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160308203236460-853216066.png)

共享的局部变量被提升为委托类的一个字段了：

- 变量i的生命周期延长了；
- for循环结束后字段i的值是5了；
- 后面再次调用委托方法，肯定就是输出5了；

那该如何修正呢？很简单，委托方法使用一个临时局部变量就OK了，不共享数据：

```c#
List<Action> acss = new List<Action>(5);
for (int i = 0; i < 5; i++)
{
    int m = i;
    acss.Add(() => { Console.WriteLine(m); });
}
acss.ForEach(ac => ac()); // 输出了 0 1 2 3 4
```

# GC与内存管理

#### 1，简述一下一个引用对象的生命周期？

#### 2，创建下面对象实例，需要申请多少内存空间？

```c#
public class User
{
    public int Age { get; set; }
    public string Name { get; set; }

    public string _Name = "123" + "abc";
    public List<string> _Names;
}
```

#### 3，什么是垃圾？

#### 4，GC是什么，简述一下GC的工作方式？

#### 5，GC进行垃圾回收时的主要流程是？

#### 6，GC在哪些情况下回进行回收工作？

#### 7， using() 语法是如何确保对象资源被释放的？如果内部出现异常依然会释放资源吗？

#### 8，解释一下C#里的析构函数？为什么有些编程建议里不推荐使用析构函数呢？

#### 9，Finalize() 和 Dispose() 之间的区别？

#### 10，Dispose和Finalize方法在何时被调用？

#### 11，NET中的托管堆中是否可能出现内存泄露的现象？

#### 12，在托管堆上创建新对象有哪几种常见方式？

#### 13，对象创建

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160309235623241-1001221060.png)

```
public class User
{
    public int Age { get; set; }
    public string Name { get; set; }

    public string _Name = "123" + "abc";
    public List<string> _Names;
}
```

- 对象大小估算，共计40个字节
  - 属性Age值类型Int，4字节；
  - 属性Name，引用类型，初始为NULL，4个字节，指向空地址；
  - 字段_Name初始赋值了，代码会被编译器优化为_Name=”123abc”。一个字符两个字节，字符串占用2×6+8（附加成员：4字节TypeHandle地址，4字节同步索引块）=20字节，总共内存大小=字符串对象20字节+_Name指向字符串的内存地址4字节=24字节；
  - 引用类型字段List<string> _Names初始默认为NULL，4个字节；
  - User对象的初始附加成员（4字节TypeHandle地址，4字节同步索引块）8个字节；

- **内存申请**： 申请44个字节的内存块，从指针NextObjPtr开始验证，空间是否足够，若不够则触发垃圾回收。
- **内存分配**： 从指针NextObjPtr处开始划分44个字节内存块。
- **对象初始化**： 首先初始化对象附加成员，再调用User对象的构造函数，对成员初始化，值类型默认初始为0，引用类型默认初始化为NULL；
- **托管堆指针后移**： 指针NextObjPtr后移44个字节。
- **返回内存地址**： 返回对象的内存地址给引用变量。

#### 14，GC回收

首先，需要再次强调一下托管堆内存的结构，如下图，很明确的表明了，只有GC堆才是GC的管辖区域，关于加载堆在前面文中有提到过。GC堆里面为了提高内存管理效率等因素，有分成多个部分，其中 两个主要部分：

- 0/1/2代：代龄（Generation）在后面有专门说到；
- 大对象堆(Large Object Heap)，大于85000字节的大对象会分配到这个区域，这个区域的主要特点就是：不会轻易被回收；就是回收了也不会被压缩（因为对象太大，移动复制的成本太高了）；

![image](https://images2015.cnblogs.com/blog/151257/201603/151257-20160309235624382-1396676769.png)





