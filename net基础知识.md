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

#### 2，在字符串连接处理中，最好采用什么方式，理由是什么？

#### 3，使用 StringBuilder时，需要注意些什么问题？

#### 4，以下代码执行后内存中会存在多少个字符串？分别是什么？输出结果是什么？为什么呢？

```c#
string st1 = "123" + "abc";
string st2 = "123abc";
Console.WriteLine(st1 == st2);
Console.WriteLine(System.Object.ReferenceEquals(st1, st2));
```

#### 5，以下代码执行后内存中会存在多少个字符串？分别是什么？输出结果是什么？为什么呢？

```c#
string s1 = "123";
string s2 = s1 + "abc";
string s3 = "123abc";
Console.WriteLine(s2 == s3);
Console.WriteLine(System.Object.ReferenceEquals(s2, s3));
```

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

















