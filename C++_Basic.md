# 1，C++基础

**（1）头文件的查找顺序**

```c++
//<>表示从编译器默认的路径中寻找，使用c++标准库的头文件使用该方式
//“”表示从当前目录下寻找文件，如果找不到再从编译器默认的路径中查找使用用户自定义的头文件使用此方式。
#include <MMSystem.h> 
#include "MyCommon.h"
```

**（3）std是个命令空间，cout是空间中的对象，endl也是对象，是回车符**

```c++
namespace China{
    float population = 14.1;
    std::string capital = "beijing";
}

namespace Japan{
    float population = 1.1;
    std::string capital = "tokyo";
}

using namespace Japan;

int main()
{
    std::cout <<"captial:"<<captial<<std::endl;
     std::cout <<"captial:"<<population<<std::endl;
}

```

**（4）看 const 离类型(int)近，还是离指针变量名近，离谁近，就修饰谁，谁就 不能变**

```c++
//第三种 暖男型 
int * const nuan_nan = &wife; 
*nuan_nan = 26;
printf("暖男老婆的年龄：%d\n", wife);
//nuan_nan = &girl; 
//不允许指向别的地址 
//第四种超级暖男型的 
const int * const super_nuan_nan = &wife; 
//不允许指向别的地址， 不能修改指向变量的值 
//*super_nuan_nan = 28; 
//super_nuan_nan = &girl;
```

**（5）p++ 的概念是在 p 当前地址的基础上 ,自增 p 对应类型的大小, 也就是说 p = p+ 1*(sizeof(类型))**

**（6）二级指针**

```c++
int guizi1 = 888; 
int *guizi2 = &guizi1; //1 级指针，保存 guizi1 的地址
int **liujian = &guizi2; //2 级指针，保存 guizi2 的地址，guizi2 本身是一个一级指针变量
```

**（7）引用和指针的区别**

```c++
int age=30;
int &ageRef = age;
int * agePtr = &age;
```

**（8）虚函数和多态**

```c++
class Person{
    public:
    	virtual void run(){
            cout<<"person run"<<endl;
        }
}

class Student: public Person{
public:
	void run(){
        cout<<"student run"<<endl;
    }    
}

int Test{
    Person* person = new Student();
    person->run();
}
```

**（9）函数的参数，局部变量在内存中怎么布局的**

```c++
int sum(int a,int b){
    static int c = 1;
    int d =a+b;
    return d;
}
```

**（10）形参和实参**

```c++
void test(int b)//b是形参数
{
    b = 9;
}

int main()
{
    int a =10;
    test(a);//a是参数，有实际的地址存储a的值
    std::cout<<a;
}

// 不推荐
if (a == 0) {
}

// 推荐
if (0 == a) {
}
```

**（11）预处理命令**

```c++
#define PI 3.14
/*
.
*/
#undef PI
#define R  3.0
#define PI 3.14
#define L  2*PI*R
#define S  PI*R*R

//带参数的宏替换
#define average(a, b) (a+b)/2
 
 int main () {
     int a = average(10, 4);
      
      printf("平均值：%d", a);
      return 0;
 }
//宏和函数的区别
//宏定义不涉及存储空间的分配、参数类型匹配、参数传递、返回值问题
//函数调用在程序运行时执行，而宏替换只在编译预处理阶段进行。所以带参数的宏比函数具有更高的执行效率
```

**（12）数组**

```c++
{
//正确的写法
int  a[5];   // 整型常量
int  b['A'];  // 字符常量，其实就是65
int  c[3*4];  // 整型常量表达式
}
{
//错误的写法
int a[]; // 没有指定元素个数，错误

int i = 9;
int a[i]; // 用变量做元素个数，错误
    
}
{
    int a[4] = {2, 5};//默认的后面为0，也就是a[2]=0,a[3]
}
```

**（13）内存操作**

```c++
int a =10;
int b = a+2;
//a和b存放在内存
//mov eax a 把a变量的值移到寄存器eax中
//add eax,2 cpu运算器执行加法操作，并把结果回写到eax中
//mov b,eax 把eax的结果移到内存b变量中
```

**（14）汇编和机器**

```c++
//汇编语言和机器语言是一一对应的
//但是汇编几乎不可能还原成高级语言
```

**（15）函数重载**

```c++
//c语言不支持函数重载,c++支持
//函数类型不同，个数不同，顺序不同

int sum(int v1,int v2)
{
    return v1+v2;
}

int sum(int v1,int v2, int v3)
{
    return v1+v2+v3;
}

int sum(int v1,float v2)
{
    return 0;
}

//返回值的类型不能不同，这样不满足函数重载，因为会有歧义
int sum()
{return 0;}
float sum()
{return 0;}

//隐式转换带来的歧义，因为此时传入整数值10，都可以满足long和double类型
void display(int a){}
void display(long a){}
void display(double a){}

display(10); //此时会报错

//编译器采用了naming decoration的技术进行处理
//也就是在编译后，上面的display会修饰为display_int(),display_double(),display_long()等名字,不同平台编译器有一定的区别

 display(10);
00D71898  push        0Ah  
00D7189A  call        00D7141F  //函数地址不同，代表函数重载了不同的函数
00D7189F  add         esp,4  
    display(10L);
00D718A2  push        0Ah  
00D718A4  call        00D7141A   //函数地址不同，代表函数重载了不同的函数
00D718A9  add         esp,4  
    display(10.0);
00D718AC  sub         esp,8  
00D718AF  movsd       xmm0,mmword ptr ds:[00D79BD8h]  
00D718B7  movsd       mmword ptr [esp],xmm0  
00D718BC  call        00D71415   //函数地址不同，代表函数重载了不同的函数
00D718C1  add         esp,8  
    
//注意：两个16进制AB =  0010 0001 代表一个字节
//进入debug和release模式生成的汇编指令还是有很多区别的
    
/**************************************************************************/

//默认参数,顺序默认为从右到左
int sum(int v1=5,int v2=6){
    return v1+v2;
}

int sum2(int v1,int v2=6){
    return v1+v2;
}

sum();

//如果声明和实现都有，那么默认参数值只能写在声明里

int age =20 ;
int sum3(int v1,int v2=age);

void test(int a){
    
}

void func(int v1,void(*p)(int) = test ){
    p(v1);
}

void main()
{
    func(20,test); // 有点委托的感觉了
    
    void(*p)(int) = test; //p是函数指针，指向test函数
    p(10);//执行test函数
    
    sum3(1,2);
}
//实现里不能写默认参数值
int sum3(int v1,int v2)
{
    return v1+v2;
}

```

**（16）默认参数**

```c++
int sum(int v1,int v2)
{
    return v1+v2;
}
int sum2(int v1,int v2=4)
{
    return v1+v2;
}

void main()
{  //汇编生成的的一样,说明默认参数等同于赋值操作
    sum(1,4); 
    sum2(1);
}

 sum(1, 4);
00072208  push        4  
0007220A  push        1  
0007220C  call        00071424  
00072211  add         esp,8  
    sum2(1);
00072214  push        4  
00072216  push        1  
00072218  call        00071429  
0007221D  add         esp,8  

```

**（17）extern _C****  

```c++
//被extern “C” 修饰的代码会按照c语言的方式去编译


extern "C" void func(){}

extern "C" void func(int v){}
//上面会导致无法重载具有外部 "C" 链接的函数，因为c不支持函数重载
extern "C"{
    void func(){}
    void func(int v){}
}

void func();
//extern 只能修饰函数的声明
extern "C" {
    void func();
    void func(int v);
}

void main(){
    func(); //调用的是c++
    func(10);//调用的是c
}

void func(){}
void func(int v){}

//引用外部C库
extern "C" {
    #include "thirdlib.h"
}

//可以在头文件，通过ifdef来条件编译，根据c++的宏
#ifdef __cpluscplus
extern "C" {
#endif
    int sum(int v1);
    int delta(int v1);
#ifdef __cpluscplus
}
#endif

//防止头文件重复包含,针对整个文件
#pragma once 

```

**（18）内联函数 (inline function)**

```c++
inline void func(){
    cout<<"func"<<endl;
}

void main(){
    func();
    func();
}

//内联函数的话会增大main的体积，因为会复制两份代码,不会开辟栈空间，性能好一点，不存在函数调用
//普通函数调用只会保持一份func代码，而且调用时候会不断的开辟和回收栈空间

//运用场景：函数体积不大，经常调用的功能
//递归函数不会被编译器变成内联函数

//宏和内联函数的区别
//内联函数有语法检测和函数特性，传参

#define add(v) v+v

inline int add(int v){
	return v+v;    
}

int main(){
    int a =10;
    int c = add(++a);   //这种对于宏来说是不满足要求的，不具备传参的功能 11+12=23
    int c = add(++a); //内联函数为11+11 =22
    
}
```

**（19）表达式**

```c++
int a = 1;
int b = 2;
(a=b) =3;   //赋值给了a ，相当于分为两步，a=b;a=3;
(a<b?a:b)=4 //赋值给了b
```

**（20）指针**

```c++
int age =10;
int height = 30;
const int *p1 =&age; //*p1是常量
int const *p2 =&age;//*p2是常量
int * const p3 =&age;//p3是常量，但是*p3 不是常量
const int * const p4 =&age;//都是常量
int const * const p5 =&age; //同上

int *p3 = &age;
*p3 = 20; //age =20
p3 = &height;
*p3 = 40; //height =40

```

**（21）引用**

```c++
//指针的写法会比较麻烦
void swap2(int *v1,int *v2){
    int temp = 0;
    temp = *v1;
    *v1 = *v2;
    *v2 = temp;
}


//定义了两个引用
void swap(int &v1,int &v2){
    int temp = 0;
    temp = v1;
    v1 = v2;
    v2 = temp;
}

int main(){
    //引用相当于变量的别名
    int age = 10;
    //定义一个age的引用，refAge相当于是age的别名
    int& refAge = age;
    cout << refAge<<endl; //类似*p   int* p = age;
    cout << &refAge << endl; //类似p
    //引用在定义的时候必须初始化，一旦指向一个变量，就不可以再改变
    
    //可以用另一个引用初始化一个引用，相当于有多个初始值,相当于一个变量有多个别名
    int &ref1 = ref;
    int &ref2 = ref1;
    
  	int a = 1;
    int b = 2;
    //swap(int &v1=a,int &v2=b)
    swap(a, b);
    //指针的写法比较麻烦
    swap2(&a,&b)
    cout << a << endl;//2
    cout << b << endl;//1
    
}
//引用的本质就是指针，只是编译器对其进行了功能弱化和限制

//X64:64bit = 8个字节
{
    int age =10;
    int *p = &age;
    int &ref =age;
    cout<<sizeof(p);  //8个字节，指针大小就是8字节
    cout<<sizeof(ref); //4,其实就是age的变量大小
    cout<<sizeof(&ref); //8个字节，是age的指针大小
}
/****************************************************************************/
{
        int* p = &age;
006A1AE9  lea         eax,[ebp-0Ch]  
006A1AEC  mov         dword ptr [ebp-18h],eax  
    *p = 30;
006A1AEF  mov         eax,dword ptr [ebp-18h]  
006A1AF2  mov         dword ptr [eax],1Eh  

    //int& refAge = age;
    //refAge = 30;
    
}
{
     //int* p = &age;
    //*p = 30;

    int& refAge = age;
00ED1AE9  lea         eax,[ebp-0Ch]  
00ED1AEC  mov         dword ptr [ebp-18h],eax  
    refAge = 30;
00ED1AEF  mov         eax,dword ptr [ebp-18h]  
00ED1AF2  mov         dword ptr [eax],1Eh  
}
//通过上面的汇编代码可以发现引用的本质就是指针，只是编译器进行了封装和限制，为了让写法更友好
```

（22）类和结构体

```c++
//结构体默认是public，类是private
void test() {
    cout << "test"<<endl;
}

struct Person1 {
    int year;
    void (*run)() = test;//用函数指针来模拟一个类
};

//peron对象为4个字节，因为只有一个int 成员
class Person {
public:
    int m_age;
    void run() {
        cout << "Person run"<<endl;
    }
};

int main(){
    Person1 person1;
    person1.run();//调用test方法
    
    Person person;
    person.m_age = 20;
    person.run();

    Person* p = &person;
    p->run();//指针访问方式
}
//上面的指针p，person对象的内存都在栈空间，自动分配和回收

    Person person1;
    person1.m_age = 10;
    person1.run();


    Person person2;
    person2.m_age = 20;
    person2.run();
//内存分析，person1和person2空间只有成员变量，没有成员函数开辟空间，因为函数没必要每创建一个对象就生成一份函数空间，因为函数没有必要，只需要调用就行，操作逻辑都是一样的

//全局区 ，数据段
Person person;
int main(){
    
}
```

（23）内存分布

```c++
栈区
堆区
代码区
全局区
 
class Person{
    int m_age;
    //this 指针存储着函数调用者的地址
    void run(){
     	cout<<"run:"<<this.m_age<<endl;   
    }
}
    
    
int main(){
    Person person1;
    person1.m_age = 20;
    person1.run();//代码区代码访问栈区的变量值，通过this指针访问到栈区对象的地址，这是隐式传参
}

	func();
005224F2  call        00521433  //如果直接调用函数就只有一个call

    Person person1;
    person1.m_age = 10;
005224F7  mov         dword ptr [ebp-0Ch],0Ah  //10赋值给对象内存
    person1.run();
005224FE  lea         ecx,[ebp-0Ch]  //但是调用成员函数时候多了一步，把内存地址传给了ecx
00522501  call        00521361  

//进入 成员函数call
00B25E3F  pop         ecx    //弹出ebp-0Ch地址值，此地址是person1对象地址
00B25E40  mov         dword ptr [ebp-8],ecx  //返存到内存地址ebp-8
//00B25E43  mov         ecx,0B2E026h  
//00B25E48  call        00B21276  
        this->m_age=3 ;
00B25E4D  mov         eax,dword ptr [ebp-8]  //把ebp-8地址的值赋给eax，eax=ebp-0Ch
00B25E50  mov         dword ptr [eax],3  //eax=ebp-0Ch,也就是对象内存地址赋值为3
    

    Person* p1 = &person;
00CB1892  lea         eax,[ebp-0Ch]  //ebp-0Ch是person的地址值
00CB1895  mov         dword ptr [ebp-18h],eax  //ebp-18h是 p1的地址值
    p1->m_age = 20;
00CB1898  mov         eax,dword ptr [ebp-18h]  //eax是地址person的地址
00CB189B  mov         dword ptr [eax],14h  
    p1->run();
00CB18A1  mov         ecx,dword ptr [ebp-18h]  //ebp-18h是 p1的地址值，内容是person内存地址
00CB18A4  call        00CB1361  
    
    
    
//为什么要通过指针访问对象，因为系统有些数据只能通过指针访问，不是所有数据变量都有引用

//函数放在代码区，是只读的，当读到函数指令时，如果是局部变量，就会去栈区开辟空间
  
/****************************************************************************/
    
    
//1,为了能自由的控制内存的生命周期，大小，一般使用堆区
//eg：植物大战僵尸中的僵尸，这个对象不能当到栈区，因为栈区是系统回收和控制的，也不能放到全局区，这样就没法销毁

{
    void test(){
        int *p = (int *)malloc(4);   
        //指针存放在栈空间，4个字节
        //开辟的空间在堆空间，4个字节
        *p=10;
        free(p);//释放堆空间
    } 
    void test2(){
        int *p = new int; //c++写法
        *p=10;
        delete p;
        
        char * p2  = new char[4];
        delete[] p2;           
    }
    //32bit
    int main(){
		test();
    }
    
}
{
    //为一段连续的堆空间赋初始值
   	int size = sizeof(int) * 10;
    int* p = (int*) malloc(size);
    memset(p, 0, size);
    
}
{
	int main()
    {
        //栈空间创建一个对象,会访问构造函数
        Person person;

        //堆空间创建一个对象，会访问构造函数
        Person *p = new Person;
        
        //不会访问构造函数
        Person* p2 = (Person*)malloc(sizeof(Person));
    }  
}
```

**（24）构造函数&& 析构函数分析**

```c++
struct Person {
    int m_age;
    
    Person(){}
    
    ~Person(){
        
    }
};

//全局区
Person g_p1;

int main()
{
    //栈区（成员变量不会被初始化）
    Person p1;

    //堆空间，有（）成员变量就初始化
    Person* p2 = new Person;  //成员变量不会被初始化
    Person* p3 = new Person(); //成员变量初始化为0
    Person* p4 = new Person[3];//成员变量不会被初始化
    Person* p5 = new Person[3](); //3个person对象的成员变量都被初始化为0
    Person* p6 = new Person[3]{};//3个person对象的成员变量都被初始化为0

    return 0;
}

{
    struct Person{
        int m_age;
        char *m_name;
        Person(){
            memset(this,0,sizeof(Person));//当此对象内存空间值置为0
        }
    }
    
}
//通过malloc分配的对象free的时候不会调用析构函数
//析构函数没必要区回收类内部的成员变量
{
    struct Car{
        int m_price;
    }
    
    struct Person{
        int m_age;
        Car *m_car;
        
        Person(){
            m_car = new Car();
        }
        
        ~Person(){
            delete m_car; //此时需要手动回收m_car对象,不然会发生内存泄漏（该回收的没回收）
        }
    }
    
    //区别
    struct Person{
        int m_age;
        Car m_car; //这个对象直接就在此对象中
    }
} 

```

**（25）命名空间**

```c++
#include <stdio.h>
using namespace std   //向很多系统的库都是包含在std空间中的
    
void func(){
   cout<<"func()";
}

namespace KK{
    void func(){cout<<"KK::func()";}
}

int main(){
    func();//这个func不在std空间中，能调用只是因为在此文件中
    using namespace KK
    KK::func();    //这个代表使用KK空间中的func函数;"KK::func()"
    ::func();  //这个调用的就是默认全局空间中的func(),"func()"
    
}

```

 **（26）对象**

```c++
//父类和子类按顺序排放成员变量
{
    struct Person{
        public:
        	int m_age;
    }
    //私有继承，等同于private,取其中的最小值
    struct Student : private Person{
        void run(){
            m_age = 10;//取类和属性的权限最小值
        }
    }
    
}
```

**（27）初始化列表**

```c++
struct Person{
    int m_age;
    int m_height;
    //等同于，效果一样
    Person(int age,int height){
        m_age = age;
        m_height = height;
    }

    Person(int age,int height):m_age(age),m_height(height){
        
    }
}

struct Person{
    int m_age;
    int m_height;
    Person(int age=0,int height=0);
}
Person::Person(int age=0,int height=0):m_age(age),m_height(height){}

/***********************************************************************/

//构造函数之间的调用，构造函数之间不能直接调，需要放到初始化列表中

struct Person{
    int m_age;
    int m_height;
    Person():Person(10,20){
        //不能直接调用Person(10,20)
        Person(10,20);//相当于创建了一个临时的person对象
    }
    Person(int age,int height){
        m_age = age;
        m_height = height;
    }
};

//父类和子类的构造析构函数执行顺序
Person:Person()
Student::Student()
Student::~Student()
Person::~Person()


```

**（28）多态**

```c++
//c++默认不支持多态，默认执行的是父类的方法,c++通过虚函数实现多态
struct Animai{
    void speak(){
        cout<<"Animai::speak"<<endl;
    }
}

struct Cat:Animal{
    void speak(){
        cout<<"Cat::speak"<<endl;
    }
}

void Miao(Animal* p){
    p->speak();
}


int main(){
   Miao(new Cat());//默认执行animal中的方法
    
}

	p->speak();//这个是覆写virtual的
00472608  mov         eax,dword ptr [p]  
0047260B  mov         edx,dword ptr [eax]  
0047260D  mov         esi,esp  
0047260F  mov         ecx,dword ptr [p]  
00472612  mov         eax,dword ptr [edx]  
00472614  call        eax    //间接call
00472616  cmp         esi,esp  
00472618  call        __RTC_CheckEsp (04712EEh)  
	p->run();				
0047261D  mov         ecx,dword ptr [p]  
00472620  call        Animal::run (04714CEh)  //直接call

void MiaoTest(Animal animal) {
	animal.speak();
	animal.run();
}

Cat cat;
MiaoTest(cat);//这个就不行，全是执行animal方法

```

**（29）虚表**

```





```

**（30）头文件防卫声明**

```c++
#ifndef _head1_
#define _head1_

#include "head2.h"
//这个等同于下面，因为此head2.h头文件已经被定义过，所以这句话直接被忽视，这样就避免头文件重复了
#ifndef _head1_
#define _head1_
int gloabal2 =8;
#endif


int global1 =5;
#endif

```

**（31）引用类型**

```c++
void func(int &a,int &b){ //形参是引用类型，修改变量值会影响到外面
    a =10;
    b =20;
}

//c# 的值类型和引用类型，涉及到装箱和拆箱
object objValue = 4; //装箱
int value = (int)objValue; //拆箱

//下面的代码不要写，会出现问题，不要给const变量赋值引用类型
    const int a = 10;
    int& ra = (int &)a;
    ra = 8;

/*********************************************************/
int v[]{1,2,3,4,5,6};
for(auto x :v){//这里面是把数组的每个元素分别拷贝到变量x中进行遍历
    cout<<x<<endl;  
}
for(auto &rx:v){//这个相当于给数组每个元素搞了一个别名引用，省了拷贝动作，效率变高
    cout<<rx<<endl;
}

```

（31）内存分配

```c++
int* myint = new int; //分配一个int大小的内存空间
int* myint2 = new int(18) //初始值为18
if(myint !=NULL){
    *myint = 8;
    delete myint;
}

int * pa = new int[100];//开辟大小为100的整型数组空间
if(pa!=NULL){
    int*q = pa;
    *q++ = 12; //[0]=12
    *q++ = 18; //[1]=18  ,此时q已经指向[2]
    cout<<*pa<<endl; //12
    cout<<*(pa+1)<<endl; //18
    delete[] pa; //释放内存
}

char *p=NULL; //NULL实际为0
char *q = nullptr; //此时q为空指针
//使用nullptr能够避免在整数和指针之间发生混淆
cout<<typeid(NULL).name()<<endl; //int
cout<<typeid(nullptr).name()<<endl; //std::nullptr_t



```

（32）值传递

```c++
struct Student {
    int age;
    char name[100];
};

void func(Student stu) {
    stu.age = 20;//&stu	0x00bff840,形参的地址都不一样
    strcpy_s(stu.name, sizeof(stu.name), "jack");  
}

void rfunc(Student &refTempstu) {
    refTempstu.age = 30;//&stu	0x00bff840
    strcpy_s(refTempstu.name, sizeof(refTempstu.name), "marry");
}

void pfunc(Student *ptempStu) {
    ptempStu->age = 40;
    strcpy_s(ptempStu->name, sizeof(ptempStu->name), "sherry");
}

int main()
{
    Student stu1;
    stu1.age = 10;
    strcpy_s(stu1.name, sizeof(stu1.name), "tom");
    cout << stu1.age << endl;       //&stu1.age	    0x00bff978
    cout << stu1.name << endl;      //&stu1.name	0x00bff97c

    //值传递，没有修改原来的实参，发生内存拷贝
    func(stu1);

    cout << stu1.age << endl; //10
    cout << stu1.name << endl; //tom  

    //引用传递，不发生内存拷贝
    rfunc(stu1);
    cout << stu1.age << endl;//30
    cout << stu1.name << endl;//sherry

    //结构指针传递
    pfunc(&stu1);

    cout << stu1.age << endl;//30
    cout << stu1.name << endl;//sherry
}
```

（33）内联函数和杂合函数写法

```c++
//内联函数在编译的时候直接替换函数体，这比调用函数，需要执行压栈出栈的效率高
//但是做不做还是在于编译器自己决定
//内联函数的定义就要放到头文件中，这样编译器会在编译阶段进行函数体的替换

int* func(){
    int val=10;
    return &val;//操作完成，此内存会被回收
}
int main(){
    int* p =func();
    *p=11; //这种操作有隐患，此时此地址应该被系统回收了，因为上面的func已经做完了
}

```

（34）const

```c++
    char str[] = "I Love You";
    const char* ptr = str;
    *ptr = "Y"; //这种写法不对，不能修改，通过p来指向的值不能改
    str[1] = 'Y';//这可以改，因为不是通过p指针修改的

    int i = 100;
    const int &a = i; //代表a的值不能改
    i = 101;//可以修改
    a = 102; //错误
```

（35）string

```c++
int num=6;
string str(num,'a');//aaaaaa
string s1;//string s1="";

for (auto& c : str) { //引用修改
        c= toupper(c);
        //cout << c << endl;
    }
cout<<str<<endl;
```

（36）vector

```c++
//vector内不能装引用类型
vector<int&> vjs; //不允许
vector<int> v1;
vector<int>::iterator iter ;
iter.begin();//指向第一个元素
iter.end(); //指向最后一个的下一s个，不存在的元素
for(vector<int>::iterator iter = v1.begin();iter!=v1.end();++iter){
    cout<<*iter<<endl;
}

//等同于上面
for(auto beg = vec.beginI();beg!=vec.end();beg++{
	cout<<*beg<<endl;
}

```

（37）拷贝构造函数

```c++
//对象拷贝时，没有调用类的构造函数，而是使用了拷贝构造函数




```

（38）隐式转换和explicit

```c++
//编译系统在私下做了很多事情
Time myTime = 14;
Time myTime2 = {1,2,3,4,5};

//explicit不让启动隐式类型转换
	Timer timer(1,2,3);
	Timer timer2 = { 1,2,3 }; //这个不行了，不支持隐式类型转换
	Timer timer2 =Timer{ 1,2,3 };//改成这种就行
//构造函数初始化列表

class Timer
{
public:
	int hour;
	int minute;
	int second;
	void initTime(int temphour,int tempminute,int tempsecond);

	explicit Timer(int temphour, int tempminute, int tempsecond);

	Timer(int temphour, int tempminute);

public:
	//这种会被直接当成inline函数,如果定义在.h文件中
	void addhour(int val) {
		hour += val;
	}

};

//成员函数末尾的const
//这个函数表示不会修改成员函数中的任何变量
void Timer::modifiedhour(int temphour) const {
    //这个函数不会修改类中的变量
    //hour += 1;//直接报错
}

public:
	mutable int hour; //加上muteble，即使被const修饰也能修改
	int minute;
	int second;

```

（39）static

```c++
//static 成员变量，此变量不属于多个对象，属于类，任何一个对象对其变更，其他对象都可以看到



```

（40）友元类

```c++
class A{
private:
    int data;
    friend class C;
};

class C{
public:
    void callA(int x,A &a){
        a.data = x;//正常情况这种不行，加上friend C后就可以访问了
    }

}
//友元没有传递性
```

（41）函数模板

```c++
template<typename T>
T funcadd(T a,T b) {
	return a + b;
}
//和c#的泛型差不多


```



（42）静态成员变量的初始化

```c++
//静态成员变量的初始化放到外面进行
class Car{
public:
    static int m_price = 0；//错误
}
int Car::m_price = 0; //正确初始化
//对比全局变量，静态变量可以设定访问权限，达到局部共享的目的
```

（43）析构函数

```c++
	Car car(3, 4); //调用析构函数
	Car* p_car = new Car(1,2);// 调用析构函数
	delete(p_car);

	Car* p_car = new Car[3];
	delete[] p_car;//会调用三次析构
	delete p_car;//会调用一次析构



```













# 2，数据结构和算法



# 3，数据库开发



# 4，MFC Windows应用开发



# 5，QT跨平台开发



# 6，高性能服务器开发

（1）并发和多线程

```c++
//进程之间的通讯方法：
//（同一电脑上）管道，文件，消息队列，共享内存
//（不同电脑上）socket通讯

//多线程之间的通讯（共享内存空间）
//全局变量，指针，引用
//问题：数据一致性控制

//windows:CreateThread(),_beginThread()
//linux:pthread_create()
//临界区，互斥区

void thread1() {
    cout << "thread1" << endl;
}

int main()
{
    thread myjob(thread1);
    myjob.join();//阻塞主线程，等待子线程的回归
	myjob.detach();//和主线程分离,交给运行库接管
    
    cout << "main thread" << endl;

}

/**********************************************************************/
struct TA {
public:
    int &m_i;//引用会带来新问题
    TA(int &i) :m_i(i) {}//值拷贝，不是引用
    void operator()() { //操作符重载
        cout << "m_i的值为:"<<m_i<< endl;
    }
};

int main()
{
    int my_i = 9;
    TA ta(my_i);
    thread myjob(ta);
    myjob.join();
}

/**************************************************************************/
void thread2(const int &i,int * ptr) {
    cout << i << endl;
    cout << *ptr << endl;
}

int main()
{
    int val = 1;
    int age = 10;
    char str[] = "this is test";
    int& m_val = val;
    int* p = &age;


    thread job(thread2, val, str);//这种会发生内存数据问题，访问不了
    job.detach();
}

//



```

（2）多线程之间数据共享

```c++
//只读数据没有数据共享的问题，属于线程安全的
vector<int> g_val = { 1,2,3 };//共享数据,只读

void print() {
    cout << "thread id:" << std::this_thread::get_id() << g_val[0]<<g_val[1]<<g_val[2]<<endl;
}

int main(){
    vector<thread> vt;
    for (int i = 0;i < 10;i++) {
        vt.push_back(thread(print));
    }   

    for (auto iter = vt.begin();iter != vt.end();++iter) {
        iter->join();
    }  
}
/***********************************************************************/
//读写操作的控制
class A {
public:
    void inMsgRecvQueue() {
        for (int i = 0;i < 10000;i++) {
            cout << "msg received" <<i<< endl;
            msgRecvQueue.push_back(i);
        }
    }

    void outMsgRecvQueue() {
        for (int i = 0;i < 10000;i++) {
            if (!msgRecvQueue.empty()) {
                int command = msgRecvQueue.front();//获取此消息
                msgRecvQueue.pop_front();//去除此消息
            }
            else
            {
                cout << "Msg is null" << endl;
            }
        }
    }
private:
    std::list<int> msgRecvQueue;

};

int main(){
    A myobj;
    thread myOutMsgObj(&A::outMsgRecvQueue, &myobj);//读线程，引用参数才能保证是同一个对象
    thread myInMsgObj(&A::inMsgRecvQueue, &myobj);
    myOutMsgObj.join();
    myInMsgObj.join();
    //上述操作线程不安全，需要引入互斥量mutex
}

{
    //加锁解锁
      m_mutex.lock();
      msgRecvQueue.push_back(i);
      m_mutex.unlock();
}
{
    //使用lock_guard类
    lock_guard<mutex> guard(m_mutex);//构造函数实现了lock操作，析构函数实现了unlock操作
    m_mutex.lock();//不能重复使用，会抛异常
}
{
    //死锁只会诞生在多于两个信号量的情况下
    线程1：lock(A)    lock(B) Unlock(B) Unlock(A)
    线程2: 	lock(B)   lock(A)Unlock(A)Unlock(B)
					//此时发生死锁，线程1无法lockB，等待线程2UnlockB，线程2无法lockA
    //死锁解决方案：
    //（1）多个线程上锁的顺序一致就不会发生死锁
    //（2）std::lock(mutex1,mutex2,....)可以同时锁多个互斥量
   std::lock(mutex1,mutex2,....)
   .......
   m_mutex.unlock();
   m_mutex2.unlock();
        
    
}
{
     static MyCAS* GetInstance() {
         //双重检查，避免每次判断锁，只在初始化时进行锁判断
        if (mInstance == NULL) {
            std::unique_lock<std::mutex> mymutex();
            if (mInstance == NULL) {
                mInstance = new MyCAS();
            }
        }
        return mInstance;
    }    
}
```

（3）单例

```c++
class  Rocket
{
public:
	static Rocket* sharedRocket() {
		if (ms_rocket == NULL) {
			ms_rocket = new Rocket();
		}
		return ms_rocket;
	}

private:
	Rocket() {}
	static Rocket* ms_rocket;
};

Rocket* Rocket::ms_rocket = NULL;
```

（4）拷贝构造函数

```c++
//利用已经存在的对象去拷贝到新的对象时，此时需要调用拷贝构造函数，而不是普通的构造函数

class Car {
private:
	int m_price;
	int m_length;
public:
	Car(int price=0, int length=0) :m_price(price), m_length(length) {
	
	}
	void display() { 
		cout << m_price << m_length << endl;
	}
};

	Car car1(1, 2);
	Car car2(car1);//此时调用了拷贝构造函数
	Car car3 = car2;//这个不是构造。，只是简单的赋值操作
```

（5）浅拷贝和深拷贝

```c++
//这个需要从内存模型进行分析
//浅拷贝是只拷贝地址
//深拷贝是拷贝内容，并修改指向对象的地址

Car car1(1,2);
car






```



















































# 7，汇编

**（1）汇编种类**

```
//8086 （16bit）
//x86 （32bit）
//x64 （64bit） Intel(windows) && AT&T(Mac)
/arm汇编 （嵌入式，移动设备）
```

**（2）寄存器**

```
//通常，CPU会把内存中的数据存储到寄存器中，然后再对寄存器的数据进行运算
64bit
RAX\RBX\RCX\RDX：通用寄存器

32bit
EAX\EBX\ECX\EDX

16bit
AX\BX\CX\DX
```

```c++
int main(){
    //内联汇编x86支持
    __asm{
        mov eax,10;
        mov rax,1122334455667788h;//此时会修改eax的值，因为rax内包含eax
    }      
}
```

**（3）MOV**

```c++
mov [1122h],3 //中括号内部放的是内存地址
//word是2字节，dword是4字节，qword是8字节
//call 函数地址
int a = 10;
00BF1FA8  mov  dword ptr [ebp-8],0Ah   //从内存地址取4个字节大小给赋值,ebp-8时变量a的地址
```

**（4）CALL**

```c++
  test();
00E218B8  call        00E21451  
    func();
00E218BD  call        00E2144C  
    
/************************************************************************/
    int tempA = 10;
//局部变量的值时动态分配的，所以用ebp控制
006B18B8  mov         dword ptr [ebp-8],0Ah  
    int b = gloabA;
//全局变量globalA在程序运行时一直不变，所以是个确定的地址值
006B18BF  mov         eax,dword ptr ds:[006BC020h]  
006B18C4  mov         dword ptr [ebp-14h],eax  
    
//CPU大小端模式，小端模式：低字节放到低地址
mov dword ptr[1123h],3
    
内存地址    内容
1122h	
1123h	0000,0011
1124h	0000,0000
1125h	0000,0000
1126h	0000,0000
1127h
1128h
1129h
```

**（5）LEA**

```c++
lea eax,[11222H]  //直接将1122H地址值赋值给eax,load effect address 装载一个有效的地址值
//eax == 1122H
//区别
mov eax, dword ptr [1122H]
//eax == 4 假设1122H地址存放的值为4

```

**（6）XOR，ADD，SUB，INC，DEC，JMP**

```c++
xor op1,op2  //op1 = op1^op2  异或
add op1,op2  //op1 = op1+op2
sub op1,op2  //op1 = op1-op2
inc op       //op = op +1
dec op       //op = op-1
jmp 内存地址   

int a = 10;
00DA2328  mov         dword ptr [ebp-8],0Ah  
    int b = 5;
00DA232F  mov         dword ptr [ebp-14h],5  
    if (a>b){
        //把a内存的值移到eax寄存器
00DA2336  mov         eax,dword ptr [ebp-8]  
    //比较寄存器eax和内存变量b的值
00DA2339  cmp         eax,dword ptr [ebp-14h]  
    //如果结果为大于就跳转,有条件跳转
00DA233C  jle         00DA2353  
        cout << "aaa";
00DA233E  push        0DA9B30h  
     
```

**（7）指针的汇编分析**

```c++
int age = 10;
int* p = &age;
*p = 5;

004B1EE2  mov         dword ptr [ebp-0Ch],0Ah  //赋值 ebp-0Ch地址，变量age
004B1EE9  lea         eax,[ebp-0Ch]  //age地址转移到eax
004B1EEC  mov         dword ptr [ebp-18h],eax  //ebp-0Ch 值放到ebp-18h内存上
004B1EEF  mov         eax,dword ptr [ebp-18h]  //ebp-0Ch 转移到eax
004B1EF2  mov         dword ptr [eax],5  // 5赋值给 ebp-0Ch地址内存，变量age
   
int& ref = age;
ref = 10;

00441EF8  lea         eax,[ebp-0Ch]  //mov不支持运算，比如mov eax,ebp-0Ch
00441EFB  mov         dword ptr [ebp-24h],eax  
00441EFE  mov         eax,dword ptr [ebp-24h]  
00441F01  mov         dword ptr [eax],0Ah  //eax的地址为ebp-0Ch，变量age
    
//由上面可知，引用本质上和指针一样    

    
//指针引用
int age =10;
int *p=&age;
int *&ref = p;//ref是p的别名
*ref = 30 // *p=30
    
//指针应用
    int age = 10;
    int* p = &age;
    int*& ref = p;//ref是p的别名
    *ref = 30;
    int height = 20;
    ref = &height; //ref 指向了height的地址
    *ref = 40;//此时age=30,height=40，ref就是p
    
/***********************************************************************/
int array[]={1,2,3};
int *p;
int *arr1[3]={p,p,p};//arr1为指针数组
int (*arr2)[3] //数组指针，等同于 int arr2[3]

//传指针可以修改变量值
void test(int *p){}
//传应用可以修改外面的变量值
void testRef(int &ref){  }
int main(){
    int a = 10;
    test(&a);
    cout << a<<endl;//1
    int& aRef = a;
    testRef(aRef);
    cout << a;//2
}

```

# 8，基础

（1）文件读写

```c++
{
    short var = 20000;
    ofstream fs;    //STL模板库中的
    fs.open("d:\\123.txt");
    fs << var;
    fs.close();

    ofstream fs2;
    fs2.open("d:\\456.txt");
    fs2.write((const char*)&var, sizeof(short));
    fs2.close();

    short value = 0;
    ifstream fi;
    fi.open("d:\\456.txt");
    fi.read((char*)&value, sizeof(short));   
}
    
```

（2）vector，lIst，map，string

```c++
//初始化	
vector<int> one;
vector<int> two(3, 10);
vector<int> three(two.begin(), two.end());
vector<int> four(three);
int myints[] = { 1,2,3,4 };
vector<int> five(myints, myints + sizeof(myints) / sizeof(int));
	
//list和map只能通过迭代器进行遍历
for (map<int, char>::iterator itor = stud_sex_map.begin();itor != stud_sex_map.end();++itor ) {
        int key = itor->first;
        char ch = itor->second;
        cout << "key=" << key << "value=" << ch << endl;
}

```

（3）虚函数

```c++
//使用基类之指针，指向派生类的对象，调用虚函数的时候，最后调用的是派生类的函数！
//一般情况下是某个函数所在的类可能会作为父类/基类，而且该函数有可能会被派生类重写，并被派生类使用，那么这个时候就可以考虑将该函数声明为 virtual 虚函数
{
    int main(){
        CZhongStudentg stud_zhong;
        stud_zhong.shangke();//调用子类方法
        
        CStudent *pStud = &stud_zhong;
        pStud->shangke();  //调用父类方法
        				//但是基类CStudent的shangke()是虚函数的时，调用的就是子类方法
    }
    
}



```



# 9，STL







# 10，Boost

```


```



# 11，服务器开发(centos7)

**（1）准备**

```c++
1，安装vim
yum install vim-gtk
2，设置网络地址
vim /etc/network/interfaces
3，gcc编译c程序，g++编译g++程序
4，解决无法共享挂载的问题
//https://www.jianshu.com/p/19a3e75327a8
5，安装gcc和g++
yum install gcc gcc-c++
```

**（2）nginx**

```c++
//linux epoll 和 windows ICOP 高并发技术
//安装pcre库，支持的正则表达
{
    # wget http://sourceforge.net/projects/pcre/files/pcre/8.32/pcre8.32.tar.gz/download
    # tar -xzvf pcre-8.32.tar.gz
    # cd pcre-8.32
    # ./configure --prefix=/usr/local/pcre
    # make && make install
}
{
    //或者直接运行
    yum install pcre pcre-devel
}


//安装zlib库 解压缩
{
    #yum install -y zlib zlib-devel
}
//安装openssl库 ssl相关
{
    #yum install openssl openssl-devel
}
//下载nginx源码
{
    wget http://nginx.org/download/nginx-1.16.1.tar.gz //上官网下载
    tar -xzvf download.zip //解压nginx
    ./configure   //nginx环境准备脚本
    make      //运行上面教程生成的makefile
        
        
}
{
    //错误解释No rule to make target `build', needed by `default'.
    //需要安装前面需要安装的库
    //https://www.cnblogs.com/karrya/p/11851496.html
    #yum -y install make zlib-devel gcc-c++ libtool openssl openssl-devel
    #./configure 
    #make && make install
}
{
    //[nginx] 在虚拟机中的 web 无法被主机访问的解决方法
    //首先开启web端口防火墙
    #firewall-cmd --permanent --add-port=80/tcp 
    //重启防火墙
    #firewall-cmd --reload
}
//master进程和worker进程通讯的方式，信号量，共享内存等
//master进程监控多个worker进程，worker进程和用户交互

//查看本机的cpu处理器数量
#grep -c processor /proc/cpuinfo
//配置nginx。conf为多核
//结果是可以看出来启动了多个worker进程
[root@localhost sbin]# ps -ef | grep nginx
root      10409      1  0 02:25 ?        00:00:00 nginx: master process ./nginx
nobody    10410  10409  0 02:25 ?        00:00:00 nginx: worker process
nobody    10411  10409  0 02:25 ?        00:00:00 nginx: worker process
nobody    10412  10409  0 02:25 ?        00:00:00 nginx: worker process
nobody    10413  10409  0 02:25 ?        00:00:00 nginx: worker process
root      10415  10327  0 02:25 pts/0    00:00:00 grep --color=auto nginx
//nginx重载配置文件
#sudo ./nginx -s reload
//linux和windows传输文件
//https://www.cnblogs.com/sixfiv/p/9771770.html （感觉不安全）
//用pscp来传输，linux上传到windows
C:\Users\Administrator\Desktop>pscp.exe -r root@192.168.61.128:/root/nginx-1.16.1.tar.gz d:\linuxshare
//windows传到linux
pscp file1.txt root@ip:/root/
//windows 批量文件上传
C:\Users\Administrator\Desktop>pscp.exe  -r D:\linuxshare\mynginx\* root@192.168.61.128:/root/mynginx
   
//让挂载生效
（1）重新安装vmware-tool
（2）挂载cdrom
[root@localhost mnt]# mkdir cdrom
[root@localhost mnt]# mount /dev/cdrom /mnt/cdrom
mount: /dev/sr0 is write-protected, mounting read-only
[root@localhost cdrom]# ls
manifest.txt     VMwareTools-10.3.10-13959562.tar.gz  vmware-tools-upgrader-64
run_upgrader.sh  vmware-tools-upgrader-32
[root@localhost cdrom]# 
[root@localhost cdrom]# 
[root@localhost cdrom]# cp VMwareTools-10.3.10-13959562.tar.gz ../
[root@localhost cdrom]# 
[root@localhost cdrom]# ls
manifest.txt     VMwareTools-10.3.10-13959562.tar.gz  vmware-tools-upgrader-64
run_upgrader.sh  vmware-tools-upgrader-32
[root@localhost cdrom]# cd ..
[root@localhost mnt]# ls
cdrom  hgfs  VMwareTools-10.3.10-13959562.tar.gz
[root@localhost mnt]# 
（3）解压
（4）运行 vmware-install.pl
    
//gcc -o XX  xx.c  生成指定的可执行文件    
//pts是虚拟终端
//关闭终端也会导致终端上启动的进程关闭
[root@localhost ~]# ps -ef | grep bash
root        627      1  0 03:33 ?        00:00:00 /bin/bash /usr/sbin/ksmtuned
root       9599   7922  0 03:33 pts/0    00:00:00 -bash
root      11035  11030  0 03:54 pts/1    00:00:00 -bash
root      11143  11035  0 03:58 pts/1    00:00:00 grep --color=auto bash
[root@localhost ~]# ps -ef | grep nginx
root      11029   9599  0 03:54 pts/0    00:00:00 ./nginx
root      11145  11035  0 03:59 pts/1    00:00:00 grep --color=auto nginx

//上面可以看到nginx的进程是从bash上fork出来的，pid号可以对应
    
```

**（3）bash启动进程流程**

```c++
init(pid=1，初始进程)------(fork)----->getty(打开终端设备0，1，2，读取用户和环境信息)-------(exec)----->login------(fork)----->bash------(fork)------>各种程序比如cd,cp,rm,mkdir,ps,vi
    
//strace 跟踪进程
[root@localhost ~]# ps -ef | grep nginx
root      11029   9599  0 03:54 pts/0    00:00:00 ./nginx
root      11453  11035  0 04:23 pts/1    00:00:00 grep --color=auto nginx
[root@localhost ~]# strace -e trace=signal -p 11029
Process 11029 attached
rt_sigprocmask(SIG_BLOCK, [CHLD], [], 8) = 0
rt_sigaction(SIGCHLD, NULL, {SIG_DFL, [], 0}, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
rt_sigprocmask(SIG_BLOCK, [CHLD], [], 8) = 0
rt_sigaction(SIGCHLD, NULL, {SIG_DFL, [], 0}, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
^CProcess 11029 detached
[root@localhost ~]# 


```

**（4）fork函数**

```c++
//ps -eo pid,ppid,sid,tty,pgrp,comm,stat | grep -E 'bash|PID|nginx'
[root@localhost ~]# ps -eo pid,ppid,sid,tty,pgrp,comm,stat | grep -E 'bash|PID|nginx'
   PID   PPID    SID TT         PGRP COMMAND         STAT
 10194  10189  10194 pts/0     10194 bash            Ss
 10642  10194  10194 pts/0     10642 nginx           S+
 10643  10642  10194 pts/0     10642 nginx           S+
 10649  10644  10649 pts/1     10649 bash            Ss
[root@localhost ~]# 
```

（5）socket

```c++
//socket是个数字，是个文件描述符，所以socket可以读写数据，接受发送数据
//netstat -anp | grep -E 'State|9000'    显示当前电脑9000端口的服务
[root@localhost cppdemo]# netstat -anp | grep -E 'State|9001'
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:9001            0.0.0.0:*               LISTEN      22098/./nginx_serve 
Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Path

//time await  tcp状态转换
>pscp.exe  -r D:\linuxshare\demo\nginx_server.cpp root@192.168.61.128:/root/cppdemo/

//listenfd是监听套接字，connfd是连接成功的套接字
connfd = accept(listenfd,(struct sockaddr*)NULL,NULL);

    
    
```

（5）epoll技术

```
//epoll支持高并发的原因是内部的数据结构很高校，是基于红黑树来存储socket和socket的事件关系，当查找socket进行响应时，红黑树的查找效率很高




```

















