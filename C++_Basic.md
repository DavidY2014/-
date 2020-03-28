# 1，C++基础

（1）头文件的查找顺序

```c++
//<>表示从编译器默认的路径中寻找，使用c++标准库的头文件使用该方式
//“”表示从当前目录下寻找文件，如果找不到再从编译器默认的路径中查找使用用户自定义的头文件使用此方式。
#include <MMSystem.h> 
#include "MyCommon.h"
```

（3）std是个命令空间，cout是空间中的对象，endl也是对象，是回车符

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

（4）看 const 离类型(int)近，还是离指针变量名近，离谁近，就修饰谁，谁就 不能变

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

（5）p++ 的概念是在 p 当前地址的基础上 ,自增 p 对应类型的大小, 也就是说 p = p+ 1*(sizeof(类型))

（6）二级指针

```c++
int guizi1 = 888; 
int *guizi2 = &guizi1; //1 级指针，保存 guizi1 的地址
int **liujian = &guizi2; //2 级指针，保存 guizi2 的地址，guizi2 本身是一个一级指针变量
```

（7）引用和指针的区别

```c++
int age=30;
int &ageRef = age;
int * agePtr = &age;
```

（8）虚函数和多态

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

（9）函数的参数，局部变量在内存中怎么布局的

```c++
int sum(int a,int b){
    static int c = 1;
    int d =a+b;
    return d;
}
```

（10）形参和实参

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

（11）预处理命令

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































# 2，数据结构和算法



# 3，数据库开发



# 4，MFC Windows应用开发



# 5，QT跨平台开发



# 6，高性能服务器开发