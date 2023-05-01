

# VSCODE+CLANG+CMAKE

+ 安装llvm+clang，在github上下载

  ![image-20230222143521301](C++.assets/image-20230222143521301.png)

+ 安装任意版本的vs，打开x64 Native Tools Command Prompt控制台，输入code打开vscode（不是必须，这样可以防止后续生成窗口程序，链接windows库的时候出现错误）

+ 安装CMake

+ 安装CMake、CMake Tools插件。

+ 安装ninja，将安装路径添加到path中。

+ 在vscode中ctrl+shift+p，打开CMake:Edit User-Local CMake Kits

  ![image-20230115185315195](C++.assets/image-20230115185315195.png)

  

+ 选择使用clang编译器

  ![image-20230115185337505](C++.assets/image-20230115185337505.png)

+ 配置：根据CMakeLists.txt文件生成目标平台下的原生工程。

  ![image-20230226160248835](C++.assets/image-20230226160248835.png)

  ![image-20230226160306474](C++.assets/image-20230226160306474.png)

+ 构建

  ![image-20230226160541376](C++.assets/image-20230226160541376.png)

  按F7，成功生成

+ 安装clangd插件

+ 配置clangd

  ```
  																																																				"clangd.arguments": [
  "--background-index",
  "--compile-commands-dir=build",  //compile_command.json相对路径，cmake默认生成在build，自行配置
  "-j=12",
  "--all-scopes-completion",
  "--completion-style=detailed",
  "--header-insertion=iwyu",
  "--pch-storage=memory",
  "--cross-file-rename",
  "--enable-config",
  "--fallback-style=WebKit",
  "--pretty",
  "--clang-tidy"
  // 网上别人配置clang++，但我这边windows、linux实测不加这行也没啥问题，可能mac可能需要另外加
  "--query-driver=clang++",
  ],
  ```

+ 下载语言服务器

  ![image-20230115185423412](C++.assets/image-20230115185423412.png)

## 编码

**ASCII**

​	ASCII将英文字母、数字、标点符号（共95个可见字符）和32个控制字符映射到0~127个字符之间。

![image-20230217153221979](C++.assets/image-20230217153221979.png)

​	字符数等于字节数

![image-20230217153938839](C++.assets/image-20230217153938839.png)

**GB2312、GBK、GB18030**

​	当中文出现在计算机上时，需要用16表示一个字符。

![image-20230219141844112](C++.assets/image-20230219141844112.png)

![image-20230219142042517](C++.assets/image-20230219142042517.png)

+0XA0是为了区分ASCII

![image-20230219145509724](C++.assets/image-20230219145509724.png)

​	随着语言越来越多，需要的编码方式越来越多，为了统一，设计了unicode。

**UNICODE**

![image-20230218113450742](C++.assets/image-20230218113450742.png)

​	unicode中不用字符（character），用字位（Grapheme，有意义的书写符号单位）。

​	用字位来表示unicode，是用代码点组合成一个字位。我们有了代码点列表，各个都有对应的数字值。下一步就是如何将它们转成对应的二进制（编码）。

![image-20230218115114948](C++.assets/image-20230218115114948.png)

​	ASCII只有一种编码策略，获取ASCII值，将其转化为一字节的二进制数

![image-20230218115615467](C++.assets/image-20230218115615467.png)

​	unicode有几种编码策略

​	**UTF-32**

​		utf-32将每个代码点值转为四字节的二进制数，所以叫utf-32。

![image-20230218120947585](C++.assets/image-20230218120947585.png)

​		这种编码的优势是每个代码点有相同大小的字节。缺点是浪费。用UFT-32编码是ASCII编码所占空间的4倍

![image-20230218121915761](C++.assets/image-20230218121915761.png)

​	**UTF-8**

​		utf-8将每个代码点映射到一到四字节之间。值较小的代码点映射到一字节，值较大占2到4字节。有些字母有相同的ASCII和utf-8编码。意味着老的ASCII程序可以读取简单的UTF-8字符。utf-8缺点是代码点尺寸和大小不等，这样通过索引定位就很难

![image-20230218124014909](C++.assets/image-20230218124014909.png)

## 程序的生成过程

+ **预处理（.c—.i）处理宏**（gcc -E main.cpp > main.i）

  + #ifndef #ifdef 条件预处理
  + #include 将头文件的内容复制到当前位置
  + #define 简单文本替换
  + 对文件进行序号标识
  + 删除 /**/ // （注释）
  + 对于#pragma保留不做处理

+ **编译（.i—.s）转换为汇编语言文件**（gcc -c main.cpp）

  对预处理后的文件，进行词法分析、语法分析、语义分析、代码优化、目标代码生成（汇编代码）

  将cpp文件转换成一种称为目标文件的中间格式，这些obj(.o)（包含机器代码的文件，以及其他我们定义的常数数据）文件可以传递到链接

  编译时的基本单位是源文件（因为头文件是以复制粘贴的方式，被粘贴到了源文件中再编译） 

+ **汇编（.s—.o）得到机器语言**

  将汇编代码转换为机器码语言文件（二进制文件）

+ **链接**

  将机器码语言文件转换为.exe,装入到内存中
  
  找到符号和函数在哪里，并把它们连接起来。（程序要有入口点，不一定是main()，否则会发生链接错误）
  
  当前编译单元所调用到的函数它的实现不一定在当前编译单元中



**程序内存分配：**

+ 代码区：存放程序的代码，即CPU执行的机器指令，一般只读。

+ 静态区（全局区）：存放静态变量和全局变量。

+ 常量区：存放常量(程序在运行的期间不能够被改变的量，例如: 10，字符串常量”abcde”， 数组的名字等)

+ 堆区：由程序员调用malloc()函数来主动申请的，需使用free()函数来释放内存。

+ 栈区：存放函数内的局部变量，形参和函数返回值。

## 	头文件与源文件

**头文件：**就是代码文件，与c/cpp文件无异，甚至可以用.txt文件来代替，只是这么做没有语法高亮。

**作用：**告诉引用此头文件的源文件有什么方法或者变量可以使用（即前置声明的作用）。尽管头文件里可以定义变量（会产生占用空间的变量实体），但是这么做很容易出现链接错误。

除了作前置声明使用，还可以向引用头文件的源文件提供自定义类型的内存布局（即结构体/类声明），这样可以使用点语法（点语法是以知道对象的内存布局为基础的，本质就是通过偏移量来访问），调用成员方法。

由于作前置声明和提供点语法支持是两种不同的作用，所以头文件也可以以此来分为两类。

1. **仅有前置声明**
2. **含结构体/类声明**



+ 头文件不参与编译，而每一个源文件自上而下独立编译

+ 头文件：类型的声明、函数的声明、宏的定义、类定义

+ 源文件：变量初始化、函数的实现

+ 常函数数在头文件中声明，在源文件中实现除了要加类名:: ，要保留const关键字。

+ 静态函数在头文件中声明，在源文件中实现除了要加类名:: ，要去掉static 关键字。

+ 虚函数数在头文件中声明，在源文件中实现除了要加类名:: ，要去掉virtual关键字。

+ “ "这种形式的头文件优先搜索当前目录下有没有这个文件，找不到再搜索系统目录。当前文件和“ ”处于同一工程且同一目录。

+ <>这中形式的头文件表示不要在当前目录下搜素，只在系统目录搜索。当前文件和<>不属于同一工程。

+ < cstdio > 可以理解为C++中的<stdio.h>，它会在变量及函数前加上std::

**C++需要声明**

![image-20230228163931108](C++.assets/image-20230228163943012.png)![image-20230228163955601](C++.assets/image-20230228163955601.png)

​	让编译器知道hello是一个函数，不是一个变量或者类的名字，也需要知道函数的参数和返回值类型，这样才能支持重载，隐式类型转换等特性，在链接的时候找到hello的定义

## extern

​	表明变量或者函数是定义在其他文件中的

1. **一个c文件需要调用另一个c文件里的变量或者函数，而不能从.h文件中调用变量**

   ![image-20220917153549172](C++.assets/image-20220917153549172.png)

   

2. **引用函数**

   如果需要调用其他.c文件中的函数，在文件中的函数声明前加extern即可，不加extern而直接声明函数也可以，因为声明全局函数默认前面带有extern

3. **extern "C"**

   C++在编译的时候为了解决函数的重载问题，会将函数名和参数联合起来生成一个中间的函数名称，而C语言则不会，因此会造成链接时找不到对应C语言函数的情况，此时C函数就需要用extern “C”进行链接指定，这告诉编译器，请保持我的名称，不要给我生成用于链接的中间函数名。

   ```c++
   //在.h文件的头上
   #ifdef __cplusplus
   #if __cplusplus
   extern "C"{
   　#endif
   　#endif /* __cplusplus */ 
   　…
   　…
   　//.h文件结束的地方
   　#ifdef __cplusplus
   　#if __cplusplus
   }
   #endif
   #endif /* __cplusplus */
   ```




**前置声明**

​	为了在不提供内存布局细节的前提下让源文件编译通过。为了只影响编译过程不影响链接过程，所以它实际不产生任何占用内存的对象

**好处：**

​	多数情况下能减少编译依赖，提升依赖速度。

+ 函数

  ```c++
  int func(int,int);
  ```

+ 变量

  ```c++
  extern int a;	//前置声明一个变量，假设某个c/cpp object file有一个int类型的a变量，链接期可以找到
  ```

+ 自定义类型

  ```c++
  struct type1;
  class type2;
  ```


![image-20230429210913080](C++.assets/image-20230429210913080.png)

![image-20230429210941328](C++.assets/image-20230429210941328.png)

# C

## 反码 补码

**有符号整数、无符号整数**

00000000表示0，10000000表示-0，同一个数有两个二进制表示，是不对的。而且原码加减运算复杂，一个数加上它的相反数也不等于0，因此产生了反码。我们可以将负数平移一下，10000000表示-1，11111111表示-128（理解反码），这样0的二进制就不重复了

**反码：**正数的反码还是原码，负数的反码就是它的原码除符号位外，取反，这样做，一个数加上它的相反数等于0，但是两个负数相加的结果还是不对，在运算时还要针对有无符号数制作两种电路板，因此产生了补码。

**补码：**正数的补码等于它的原码，负数的补码等于反码加1。10000000表示-128，11111111表示-1，



虽然64位计算机的寄存器能处理64位的整数，处理器去读写内存的时候靠的是寄存器提供的地址，因此寄存器的大小（也就是字的大小）决定了它能读写的内存大小，实际上的内存地址并没有64位

实际上地址的高16位始终和第48位一致（符号扩展），也就是虚拟地址空间只有48位。

而经过MMU映射后实际给内存的地址只有39位，因此如今的x64架构实际上只能访问**512GB内存**，如果插了超过这个大小的内存条它也不会认出来

16位计算机实际上能通过额外的段寄存器访问到20位的内存地址（1MB）

32位计算机还能通过PAE技术（物理地址扩展）访问到36位的内存地址（64GB）

64位计算机反而是因为16777216TB太大，内存地址被阉割到了39位（512GB）



## 类型位数

![image-20220530114353217](C++.assets/image-20220530114353217.png)



**字面常量：通过修饰符来确定**

+ 在数字后面加上U和L可以表示不同类型的字面常量
+ 32是 int 类型
+ 32L是 long 类型
+ 32LL是 long long 类型
+ 32U是unsigned int 类型
+ 32ULL是 unsigned long long 类型



**标准化的类型：stdint.h**

+ 实际上，尽管主流操作系统上int都是32位的，C语言标准并没有规定int 就是32位的
+ int甚至可以是16位的，只不过主流操作系统一致认为是32位而已，并不是标准所保证的
+ 为了解决不同操作系统上对类型定义混乱的问题，C语言标准引入了stdint.h这个头文件。
+ 他里面包含一系列类型别名（typedef），这些别名保证不论是什么操作系统什么架构，都是固定的大小
+ typedef char int8_t
+ typedef short int16_t
+ typedef int int32_t
+ typedef long long int64_t 
+ typedef unsigned char uint8_t
+ 这样不论操作系统对类型的定义如何混乱，这些标准化的类型都是确定的大小，这样就避免了跨平台的麻烦



**intptr_t和uintptr_t：自动随系统位数决定大小**

+ intptr_t在32位平台上等价于int32_t，在64位平台上等价于int64_t
+ uintptr在32位平台上等价于uint32_t，在64位平台上等价于uint64_t
+ size_t：表示大小的整数类型，其实等价于uintptr_t
+ ssize_t等价于intptr_t，是Unix/Linux系统特有的，Windows上不存在



**整数互转**

源类型<目的类型：有符号->符号扩展，无符号->零扩展

char<short -128 10000000->11111111 10000000

​					  127 01111111->00000000 01111111



**浮点数的二进制表示**

+ float由4个字节组成 ，也就是32位
+ 最高位是符号位，接着是8位指数位（e）
+ 剩下的23位是底数位（m）
+ 值得注意的是指数位（e）是+127以后表示的
+ 浮点数实际表示的值是  &plusmn;1.mmmmmmm 2^e

![image-20220530193657619](C++.assets/image-20220530193657619.png)



**abs()**

![](C++.assets/image-20220530225542062.png)

![](C++.assets/image-20220530225322865.png)

​	printf不知道参数是什么类型，会误认为是float参数，直接把int类型的4个字节推到栈上作为printf的参数，而printf会把这4个字节作为浮点数来处理，由于浮点数的指数位在高位，但整数是3，导致高位都是0，所以printf误读出来的float会是一个很小的数

cmath和math.h的区别（其他头文件类似）：cmath会有std::，是C++版本的math.h，例如上面的图片中，想要用C++有重载的abs()，需要头文件cmath



一般再栈上不能定义动态数组（gcc编译器可以，msvc不行）

![image-20220531133534749](C++.assets/image-20220531133534749.png)![image-20220531133546467](C++.assets/image-20220531133546467.png)



## 字符串

```c++
int main()
{
    // char *str = "asdasdasd"         
    //*(str+1) = 'c';				 //指向字符常量区的指针不能修改
    
    char *str = new char [10];
    //str = "asdasdasd";             //str由指向堆区变为指向字符常量区 
    strcpy_s(str,10,"aaaaaaaaa");
    std::cout << str << std::endl;
    delete[] str;                    //不能删除字符常量区
    str = 0;
    return 0;
}
```

```c++
char str[10] = "asdasdasd";		//在栈区定义一个字符数组，然后将字符常量区的字符串拷贝过来，可以指定数组长度，也可以不指									  定，字符数组只有在定义时才能将整个字符串拷贝过来
//str = "aaaaaaaaa";			//str为地址常量不能赋值
strcpy_S(str,10,"aaaaaaaaa");
```



**strcpy memcpy区别**

```c++
char *strcpy(char *_Destination, const char *_Source);		//仅用于字符串的复制
void *memcpy(void *_Dst, const void *_Src, size_t _Size);	//用于一般内存的复制，字符数组、结构体、类等
```

1. 复制的内容不同。strcpy只能复制字符串，而memcpy可以复制任意内容，例如字符数组、整型、结构体、类等。
2. 复制的方法不同。strcpy不需要指定长度，它遇到被复制字符的串结束符"\0"才结束，所以容易溢出。memcpy则是根据其第3个参数决定复制的长度。
3. 用途不同。通常在复制字符串时用strcpy，而需要复制其他类型数据时则一般用memcpy
   





**namespace** 

​	命名空间主要用于区分同一个作用域下的相同成员。

```c++
//定义命名空间
namespace A{
    namespace functions{
        void print(const std::string &text)
        {
            std::cout << text << std::endl;
        }
    }
    void print_a() {}
}
namespace B{
    void print(const char *text)
    {
        std::string temp = text;
        std::reverse(temp.begin(),temp.end());
        std::cout << temp << std::endl;
    }
}
int main()
{
    //使用命名空间的两种方法
    //using namespace A::functions;
    namespace a = A::functions;     //给命名空间取别名
    // using namespace A;
    // using namespace B;
    a::print("hello");
    A::print_a();
}
```



**define**

**缺点：没有类型检查**

```c++
// \代表和下一行连接
// 宏的参数也是替换
// ##是连接符
// #代表字符串

#define A(ThisClass)\
	ThisClass ps##ThisClass;\
	ps##ThisClass.show();

#define B(ThisClass) std::cout<<#ThisClass<<std::endl;
int main()
{
	A(CPerson);
	B(CPerson);
	system("pause");
}
```

```c
#define mul(X,Y) X*Y
printf("%d\n",mul(6+6,6+6));
//输出48而不是144,说明只是简单的机械替换,应该把参数加上括号
mul(X,Y) (X)*(Y);

#define add(X,Y) (X)+(Y);
printf("%d\n",10*add(6,6));
//输出66而不是120,因该把整个宏定义加上括号
add(X,Y) ((X)+(Y));
```

union

+ 成员变量是共享同一块内存的。找到联合体中最大的数据类型，还必须满足是所有数据成员的整数倍（字节对齐，每四个字节对齐）

  ![image-20230307154742479](C++.assets/image-20230307154742479.png)

+ 修改一个成员的值会影响其他成员

  ![image-20221001201938762](C++.assets/image-20230307154542933.png)







# C++

## new

​	动态分配内存（堆区）

​	指针变量 = new 数据类型

```c++
//定义指针变量，去掉*和指针变量名，剩下的是指针要操作的数据类型
//new 整形指针
int * (*p) = new int*;
delete p;
p = 0;
//new 整形指针数组
int *(*p1) = new int*[3];
delete[] p1;
p1 = 0;
//new 整形二维数组
int (*p2)[3] = new int[2][3];
delete[] p2;
p2 = 0;
//new 数组指针
int (**p3)[3] = new (int (*)[3]);

int *p1 =new int;
int *p2 =new int(10); //申请空间并初始化

int *p1 =new int[10];
int *p2 =new int[10](); //新建int类型数组并初始化，每个元素为0
char *p3 =new char[10](); //新建char类型数组并初始化，每个元素为空字符'\0'
```

**new malloc区别**

1. new、delete是关键字，需要C++编译器的支持，malloc()、free()是函数，需要头文件支持
2. new申请空间不需要指定申请大小，根据类型自动计算，new返回的是申请类型的地址，不需要强转，malloc()需要显示的指定申请空间大小（字节），返回void*，需要强转成我们需要的类型
3. new申请空间的同时可以设置初始化，而malloc需要手动赋值
4. new申请类、结构体对象内存空间会自动调用构造函数，delete会自动调用类、结构体的析构函数，单独的malloc()和free()则不会调用构造、析构函数。

## **inline**

**C++98之前**

+ 宏在预编译阶段被替换，内联函数在编译阶段被替换
+ inline函数是给编译器的一个建议，而不是必须
+ 牺牲空间换取的时间，（提高函数调用的效率）
+ 函数体代码量少并且逻辑简单（没有for、while）递归等

```c++
inline int add(int a,int b)
{
    return a+b;
}
```



**C++11之后**

​	内联命名空间的名称是可选的。如果未指定名称，则使用外层命名空间的名称作为内联命名空间的名称

+ 内联命名空间中的所有声明都可以直接访问，而不需要使用命名空间限定符。

+ inline namespace可以将这个namespace内嵌在另一个namespace中，并使内部命名空间的成员可与直接被外层命名空间使用。可以把一个子命名空间内的函数和类型导出到父命名空间中

  ![image-20230302150124606](C++.assets/image-20230302150124606.png)![image-20230302151749307](C++.assets/image-20230302151749307.png)

  ![image-20230302222940684](C++.assets/image-20230302222940684.png)

+ inline修饰namespace时可以在库的版本控制上提供语言上的支持。

+ ![image-20230302000701650](C++.assets/image-20230302000701650.png)



+ 上述有些情况可以通过在namespace中嵌套using namespace来替代inline namespace，但在下列两中情况中只能用inline namespace。

  ![image-20230303153319745](C++.assets/image-20230303153319745.png)![image-20230303153333184](C++.assets/image-20230303153333184.png)

+ ADL(Argument-Dependent Lookup)：扩展命名空间的查找范围，通过函数参数查找函数的命名空间。如果在函数的名称空间中定义了一种或多种参数类型，则不必为函数限定名称空间。

  ![image-20230303172231745](C++.assets/image-20230303172231745.png)

  ![image-20230303173008768](C++.assets/image-20230303173008768.png)

  ```c++
  std::swap(obj1,obj2);
  //using std::swap;
  //swap(obj1, obj2);
  //如果存在命名空间A，并且存在A::obj1,A::obj2和A::swap()，则第二个示例将导致对A::swap()的调用，这可能不是用户想要的。
  ```

**C++17之后**

+ 程序中可以有多个内联函数或变量的定义 (C++17 起)，只要每个定义出现在不同的翻译单元中，并且所有定义都是相同的。例如，一个内联函数或一个内联变量 (C++17 起)可以在包含在多个源文件中的头文件中定义。
+ 所有函数定义中的函数局部静态对象在所有翻译单元之间共享（它们都引用一个翻译单元中定义的同一个对象）
+ func在符号表中的绑定类型是GLOBAL，是全局的，bar1.cpp和bar2.cpp中各有一个GLOBAL绑定类型，在链接时func有多个GLOBAL绑定类型。GLOBAL绑定类型全局只允许有一个，链接时会发生错误。
+ 在函数前面加上inline，func的绑定类型变为WEAK，n的绑定类型也变为WEAK。在链接时如果遇到多个WEAK，会随机选择一个，所以链接可以成功。

![image-20230228164446399](C++.assets/image-20230228164446399.png)

![image-20230228164524285](C++.assets/image-20230228164524285.png)![image-20230228164600633](C++.assets/image-20230228164600633.png)![image-20230228164655074](C++.assets/image-20230228164655074.png)

![image-20230228164711375](C++.assets/image-20230228164711375.png)

![image-20230228164751133](C++.assets/image-20230228164751133.png)



+ 在类或者结构体内的静态变量只能声明不能初始化，可以通过inline修饰变量解决。

![image-20230228220152372](C++.assets/image-20230228220152372.png)

![image-20230228220234249](C++.assets/image-20230228220234249.png)

![image-20230228220305888](C++.assets/image-20230228220305888.png)

+ 成员函数函数通常是隐式inline函数。

+ 函数模板可以不用inline修饰，虽然函数模板的头文件通常会被多个编译单元包含，但是函数模板在符号表中绑定类型是WEAK，所以不会报错。

+ 如果你实现一个函数模板，而需要此模版实例化的所有函数都是inline的，那么将其声明成inline。但如果你将函数实现成模板，而此函数不需要inline，那么避免将模板声明成inline。 

+ Derived() 并不是inline函数（代码很复杂）。

  ![image-20230303202705702](C++.assets/image-20230303202705702.png)

  ![image-20230303202857192](C++.assets/image-20230303202857192.png)



## **const**

+ **常量指针**（不能改变指针指向的内容）

  ![image-20230313165837239](C++.assets/image-20230313165837239.png)

+ **指针常量**（不能改变指针本身）

  ![image-20230313165844361](C++.assets/image-20230313165844361.png)



+ 既不能改变指针本身也不能改变指针指向的内容

  ![image-20230313165826286](C++.assets/image-20230313165826286.png)

+ **常函数，不能修改类中的成员属性**

  ![image-20230313165703448](C++.assets/image-20230313165703448.png)

+ **const方法中需要修改变量，用mutable来定义变量**

+ **常量对象只能调用const方法**

  ![image-20230313165538971](C++.assets/image-20230313165538971.png)

## static

+ **类内**

  + 不依赖对象，类公用的，在类外初始化，不能在.h文件初始化（重定义）要在.cpp文件初始化，而不管创建了多少个该类的对象。所有这些对象的静态数据成员都共享这一块静态存储空间。	
  + 修饰成员函数：没有this指针，可以不依赖对象调用，类公用的，不可以声明为虚函数，不能直接调用类中的非静态成员
  + **在类中使用static的原因：**
    1. 可以让所有类实例之间共享数据
    2. static与类有关

+ **类外**

  + 在类或结构体外部使用static，链接将只是在内部，只能在定义它的cpp文件可见

  + static修饰类外变量：修饰全局，是全局静态变量(当前源文件)，生命周期到程序退出时结束，(存在静态区/全局区)

    ​									修饰局部，是局部静态变量(只能在所在函数中使用)，生命周期到程序退出时结束，(存在静态区/全局区)

  + static局部变量不初始化时，默认值为0；

  + 普通局部变量不初始化时，默认值为随机值。

  + static局部变量在编译阶段，函数还未执行的时候，就已经分配了变量空间。

![image-20230313170506957](C++.assets/image-20230313170506957.png)

**local static**

```c++
class Singlaton
{
private:
	int m_num;
	static Singlaton *s_Instance;
public:
	Singlaton()
		:m_num(6) {}
	static Singlaton& Get() { return *s_Instance; }
	void Print() { std::cout << m_num << std::endl; }
};

Singlaton *Singlaton::s_Instance = new Singlaton;

int main()
{
	Singlaton::Get().Print();
	system("pause");
	return 0;
}

/*--------------------------------------------------------*/

class Singlaton
{
private:
	int m_num;
public:
	Singlaton()
		:m_num(6){}
	static Singlaton& Get() {
		static Singlaton instance;
		return instance;
	}
	void Print() {std::cout << m_num << std::endl; }
};
int main()
{
	Singlaton::Get().Print();
	system("pause");
	return 0;
}
```



**struct**

**类和结构体的区别：**类默认的访问修饰符是private，结构体默认的访问修饰符是public

**C和C++结构体的区别：**

1. C的结构体内不允许有函数存在，C++允许有内部成员函数，且允许该函数是虚函数。
2. C的结构体对内部成员变量的访问权限只能是public，而C++允许public,protected,private三种。
3. C语言的结构体是不可以继承的，C++的结构体是可以从其他的结构体或者类继承过来的。
4. C 中使用结构体需要加上 struct 关键字，或者对结构体使用 typedef 取别名，而 C++ 中可以省略 struct 关键字直接使用

**结构体对齐**

​	经过内存对齐后，CPU的内存访问速度大大提升，因为如果没有内存对齐的话，CPU在访问一个数据时可能会进行多次访问然后拼接在一起。 

+ 规则一.： 每个**成员变量**在其结构体内的**偏移量**都是**成员变量类型**的大小的倍数。

+ 规则二： 如果有**嵌套结构体**，那么内嵌结构体的第一个成员变量在外结构体中的**偏移量**，是内嵌结构体中那个数据类型大小**最大**的成员变量的倍数。

+ 规则三： **整个结构体**的大小要是这个结构体内数据类型大小**最大**的成员变量的**倍数**。如果有内嵌结构体，那么取内嵌结构体中数据类型大小最大的成员变量作为计算外结构体整体大小的依据。

```c
typedef struct TEST{
	int na;
    char cb;
    char cc;
    int nd;
    char cf;
    struct TT{
        int ng;
        long long llh;
    }tt;
    char ci;
}test;
```

![image-20220914013257895](C++.assets/image-20220914013257895.png)



**noexcept**

```c++
int f1(int x) throw();   		//no exceptions from f1:C++98 style	less optimizable
int f2(int x) noexcept;  		//no exceptions from f2:C++11 sytle	most optimizable
int f3(int x) noexcept(true);	//ditto
int f4(int x);					//less optimizable
//在为某个异常进行栈展开的时候，会依次调用当前作用域下每个局部对象的析构函数。如果是noexcept,就会优化掉这些代码
```

**对于一个函数而言：**

+ noexcept 说明符要么出现在该函数的所有声明语句和定义语句，要么一次也不出现。
+ 函数指针及该指针所指的函数必须具有一致的异常说明。
+ 在 typedef 或类型别名中则不能出现 noexcept。
+ 在成员函数中，noexcept 说明符需要跟在 const 及引用限定符之后，而在 final、override 或虚函数的 =0 之前。
+ 如果一个虚函数承诺了它不会抛出异常，则后续派生的虚函数也必须做出同样的承诺；与之相反，如果基类的虚函数允许抛出异常，则派生类的虚函数既可以抛出异常，也可以不允许抛出异常。

```c++
//如果noexcept作用于移动构造时,会对容器在移动元素时产生影响
std::vector<Widget> vw;
Widget w;
vw.push_back(w);    //如果此时capacity==size,vw会扩充容量,把元素拷贝到新的内存上
                    //拷贝过程可以调用拷贝构造,如果移动构造声明为noexcept,那么会调用移动构造
                    //因为push_back()要保证强异常安全性,即如果插入元素时抛出异常,vector的元素不应该受到影响
                    //如果直接调用移动构造,在移动元素的过程中抛出异常,元素会丢失
```

**noexcept是可以带条件的**

```c++
template<class T,std::size_t N>
void swap(T (&a)[N],T(&b)[N]) noexcept(noexcept(swap(*a,*b)));

template<class T1,class T2>
struct pair{
    //...
    void swap(pair& p) noexcept(noexcept(swap(first,p.first)) && noexcept(swap(second,p.second)));
    //...
}
```



**c++11标准规定，类的析构函数都是noexcept的，除非显式指定noexcept(false)**



**noexcept优点：**

1. **显示指定noexcept的函数，编译器会进行优化**

   在调用noexcept函数时不需要记录exception handler，所以编译器可以生成更高效的二进制代码（编译器是否优化不一定，但理论上noexcept给了编译器更过优化的机会）。另外编译器在编译一个`noexcept(false)`的函数时可能会生成很多冗余的代码，这些代码虽然只在出错的时候执行，但还是会对Instruction Cache造成影响，进而影响程序整体的性能。

2. **容器操作针对`std::move`的优化**



```c++
//在一个noexcept函数中可以调用非noexcept函数
void setup();
void cleanup();
void func() noexcept
{
    setup();
    //...
    cleanup();
}
```



**使用建议：**

1. 析构函数
2. 构造函数（普通、复制、移动），赋值运算符重载函数
3. 保证不会抛出异常的函数



**constexpr**

​	用来定义常量表达式，使指定的常量表达式获得在编译阶段计算出结果的能力，而不必等到程序运行阶段。

常量表达式：有多个(>=1)常量组成的表达式。值不会变并且在编译过程就能够得到计算结果的表达式。数组长度就是一个常量表达式。



**修饰变量**

```c++
int n;
constexpr auto arrSize1 = n;   //error! n's value not know at compilation
std::array<int,n> arr1;        //error! ditto
constexpr auto arrSize2 = 10;
std::array<int,arrSize2> arr2;
//上面的constexpr也可以替换成const,这是因为arrSize2同时满足是const变量且使用常量表达式为其初始化这两个条件,因此编译器会认定arrSize2是一个常量表达式,但是const和constexpr并不相同

const auto arrSize3 = n;    	//fine,arraySize is const copy of n
std::array<int,arrSize3> arr3;  //error! arrSize's value not known at compilation

```

**修饰函数**

该函数必须有返回值，即函数的返回值类型不能是 void。

函数在使用之前，必须有对应的定义语句，函数的使用分为“声明”和“定义”两部分，普通的函数调用只需要提前写好该函数的声明部分即可（函数的定义部分可以放在调用位置之后甚至其它文件中），但常量表达式函数在使用前，必须要有该函数的定义。

return 返回的表达式必须是常量表达式

```c++
constexpr int pow(int base,int exp) noexcept{
    //...
    return 0;
}    
constexpr auto numConds = 10;
std::array<int,pow(1,numConds)> arr4;

//C++11 constexpr函数中不能有for循环
constexpr int pow(int base,int exp) noexcept{
    return (exp == 0 ? 1 : base*pow(base,exp-1));
}
//C++14 可以有for循环
```

**修饰构造函数和成员函数**

```c++
class Point{
private:
    double x,y;
public:
    constexpr Point(double xVal,double yVal) noexcept
        : x(xVal),y(yVal) {}
    constexpr double xValue() const noexcept {return x;}
    constexpr double yValue() const noexcept {return y;}
    void setX(double newX) noexcept {x = newX;}
    void setY(double newY) noexcept {y = newY;}
    //c++11 constexpr修饰的函数默认是const修饰的函数,在C++14中没有该规则
    //constexpr void setX(double newX) noexcept {x = newX;}
    //constexpr void setY(double newY) noexcept {y = newY;}
};
constexpr
Point midpoint(const Point& p1,const Point& p2) noexcept{
    return {(p1.xValue()+p2.xValue())/2,(p2.yValue()+p2.yValue())/2};
}
constexpr Point reflection(const Point& p) noexcept
{
    Point result;
    result.setX(-p.xValue());
    result.setY(-p.yValue());
    return result;
}
constexpr Point p1(20.01,6.6);  //fine,"runs" constexpr ctor during compilation
constexpr Point p2(8.5,1.2);
constexpr Point p3(8.0,7.8);
constexpr auto mid = midpoint(p2,p3);
constexpr auto rm = reflection(mid);	//during compilation
```

C++17 类的静态constexpr成员变量默认就是内联的，const常量和类外面的constexpr变量默认内联，需要用inline修饰。

```c++
struct Bar{
    static const int number = 2001;
};
std::cout << Bar::number << std::endl;
std::cout << &Bar::number << std::endl;
```

**枚举**

**C++98的enum是非域内枚举**

```c++
enum Example : unsigned char
{
    A = 9,B = 17,C
};
int main()
{
    Example e = C;  
    std::cout << (e == 18) << std::endl;
    return 0;
}
```

**C++11枚举的关键字为enum class**

**优点：**

+ 防止命名空间污染：enum class将{ }内的变量限制在{ }作用域可见

  ```c++
  enum class color {red,blue,green};
  auto red = false;
  auto c = color::red;
  ```

+ 强类型枚举：非域内的枚举成员，可隐式地转换为广义整型，域内枚举成员不能

  ![image-20230318210659976](C++.assets/image-20230318210659976.png)

+ 支持前置声明：即不用初始化枚举成员，声明一个枚举类型

  ```c++
  enum class Color;　　
  ```


**enum在某些情况下，也有它自己的有点**

+ 假定用户的信息用元组来表示`{name,email,reputation value}`

  ```c++
  using UserInfo = std::tuple<std::string,std::string,std::size_t>;
  //当在另一个源文件中使用上述代码，会忘记tuple中每一个变量的含义，所以会用enum来记录每个变量含义的顺序
  //UserInfo uInfo;
  //auto val = std::get<1>(uInfo);
  enum UserInfoFields{uiName,uiEmail,uiReputation};
  UserInfo uInfo;
  auto val = std::get<uiEmail>(uInfo);
  ```

+ 如果用enum class，需要类型转换

  ```c++
  enum class UserInfoFields{uiName,uiEmail,uiReputation};
  UserInfo uInfo;
  auto val = std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>(uInfo);
  //可以用模板函数,将枚举成员UserInfoFields::uiEmail和std::size_t联系起来
  template<typename E>
  constexpr typename std::underlying_type<E>::type toUType(E enumerator) noexcept
  {
      return static_cast<typename std::underlying_type<E>::type>(enumerator);
  }
  auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
  ```

  

**引用**

​	引用是C++语言的一个特殊的数据类型描述（给变量起别名）

​	用于在程序的不同部分使用两个以上的变量名指向同一个地址，使得对其中人一个变量的操作实际上都是对同一个地址单元进行的

**注意：**

1. 对引用进行操作，实际上就是对被引用的变量进行操作
2. 引用不是值，不占存储空间，声明引用时，目标的存储状态不会改变
3. 引用一旦被初始化，就不能在重新引用其他空间

**引用与指针的区别：**

1. 引用定义就要初始化
2. 引用初始化后就不能在重新引用其他的空间
3. 引用不占存储空间
4. 不能给引用赋NULL

如果要修改函数传入的参数，堆区的变量用指针，栈区的变量用引用



## 类

**访问修饰符：**

+ public：在任何一个地方都可以见到
+ protected：本类和派生类中可以见到
+ private：只有本类可以见到

**继承后父类成员在子类中的访问情况                      **

+ public：public->不变	protected->不变	private->不可访问
+ protected：public->protected    protected->不变    private->不可访问
+ private：public->private    protected->private    private->不可访问
+ 无论怎么继承，除了private在派生类不能访问，其余都可以用，继承的方式不同不影响派生类，影响派生类的对象的访问

**构造函数：**

+ 创建对象的时候自动调用构造函数

+ 构造函数可以有多个，但是一个对象只能调用一个
+ 类中如果没有构造，会有一个默认的无参构造

**析构函数：**

​	对象的生命周期结束，自动调用析构函数，清理内存

+ 析构函数不允许有返回值
+ 析构函数不允许带参数
+ 一个类中只能由一个析构函数

+ 类中如果没有析构，会有一个默认的什么都不做的析构函数

**空类中默认的函数**

​	默认无参构造、析构、拷贝构造、operator=   

​	拷贝构造、operator=这两个默认的都是浅拷贝

**抽象类和接口类**

+ 含有纯虚函数的类就是抽象类
+ 所有函数都是纯虚函数的类就是接口类

**C++阻止一个类被实例化：**

+ 抽象类
+ private构造函数

**接口（纯虚函数）**

​	在基类中定义没有实现的函数，然后强制子类去实现该函数

​	创建一个类，只有未实现的方法组成，然后强制子类去实际实现它们，这就叫接口，接口中只包含未实现的方法，由于这个接口类实际上并不包含方法实现，所以并不能实例接口类

```c++
class Printable
{
public:
	virtual std::string GetClassName() = 0;
};
class EntityBase : public Printable
{
public:
	std::string GetClassName() override { return "EntityBase"; }
};
class Entity : public EntityBase
{
public:
	std::string GetClassName() override { return "Entity"; }
};
void PrintClassName(Printable *obj)
{
	std::cout << obj->GetClassName() << std::endl;
}
int main()
{
	EntityBase *p1 = new EntityBase;
	EntityBase *p2 = new Entity;
	PrintClassName(p1);
	PrintClassName(p2);
	system("pause");
	return 0;
}
```



**类的大小**

+ 空类的大小为1B,用来占位

+ 类中的成员变量是在创建对象的时候存在，每个对象都有一份

+ 类中的成员函数是在编译的时候存在，只有一份，所有对象公用

+ 类的大小与构造函数、析构函数、静态成员变量、以及其他成员函数无关，与普通成员变量、虚函数（指向虚函数表的指针，vptr，4/8字节）、继承（子类加父类）有关，虚继承会有一个指向虚基类表的指针，4/8个字节。



**this**

​	每个非static的成员函数都有一个隐藏的指针，是当前这个类的，this，用来存储调用者对象的地址	

```c++
class Entity{
private:
    int m_x;
    int m_y;
public:
    Entity(int m_x,int m_y){
        Entity *const & e = this;
        this->m_x = m_x;
        this->m_y = m_y;
    }
    void GetThis(){
        std::cout << this << std::endl;
    }
};

int main()
{
    Entity e1(6,6);
    Entity e2(3,3);
    std::cout << &e1 << std::endl;
    e1.GetThis(/*&e1*/);
    std::cout << &e2 << std::endl;
    e2.GetThis(/*&e2*/);
    return 0;
}
```



**对象的种类** 

我们最常见的就是在函数内（包含参数）定义的栈区的**局部对象**和 使用关键字 new 在**堆区申请的对象**，局部对象在函数调用完毕或遇到}时生命周期结束，内存空间自动被系统回收，但new出来 对象直到执行delete操作时生命周期结束，内存才会被释放。

 还有一种经常遇见的**全局对象**，他的生命周期比较早，在程序运行时会先调用构造函数初始化全局 对象，然后再执行main函数，直到程序退出前，要回收全局对象内存，调用其析构。

**静态全局对象** 他的声明周期与全局对象是一样，都是伴随着程序的运行开始，直到程序退出。如 果存在多个对象那么哪个对象写在前面先被定义，则先执行其构造，最后在退出时执行析构与构造 的顺序相反。局部对象亦是如此。 

**静态全局对象**与全局对象的区别不在于生命周期，而是在于作用域。 静态全局对象作用于所属源文件里，不能作用到其它源文件文件里，即被static关键字修饰过的对 象具有文件作用域。这样即使两个不同的源文件都定义了相同名字的静态全局对象，它们也是不同 的变量。而全局对象则在所有源文件中共享。 

**静态局部对象**是定义在某个函数中的对象，但它并不是程序已创建就被初始化，而是第一次执行该 函数定义对象时被初始化，以后再调用该函数并不会创建新的对象，此对象只会存在一份。如果函 数一直没有被执行，那么该静态局部对象也不会被初始化。 与静态全局对象对象相比，静态局部对象 创建的晚些，但都是在程序退出前被回收，这也就意味 着我们可以在函数外使用静态局部对象，前提是此对象已经被创建了。



**类之间的关系**

**横向**

1. 组合：直接要求包含对象对被包含对象的拥有以及包含对象与被包含对象生命期的关系（人与器官的关系，在类内定义另一个类的对象）
2. 依赖：某个对象的功能依赖于另外的某个对象，而被依赖的对象只是作为一种工具在使用，而并不持有对它的引用（人与电脑的关系，在方法中将另一个类的对象通过参数传入）
3. 关联：某个对象会长期的持有另一个对象的引用，而二者的关联往往也是相互的。它们在生命期问题上没有任何约定。被关联的对象还可以再被别的对象关联。（人与朋友的关系，在类中定义另一个类的指针）
4. 聚合：聚合是更强版本的关联，它暗示着一种所属关系以及生命期关系。被聚合的对象还可以在被别的对象关联。（家庭和人的关系）

**纵向：继承（泛化）**

+ 继承的优点：提高了代码的复用性

+ 继承关系中，同名的成员要类名::区分

+ 继承关系中，执行父类的指定构造函数要在初始化列表中调用



子类和父类如果由同名的成员，定义子类对象，默认使用的是子类成员，son.CSon::成员，如果想使用父类的成员，son.CFather::

定义子类对象，在内存中不仅包含子类的成员，也包含父类的成员属性，内存空间首地址开始是父类成员属性，后面是子类的成员属性

## 函数

​	函数是为了让代码不重复，频繁调用函数会让程序速度变慢（编译器会生成一个call指令，创建一个堆栈结构，跳到二进制文件的不同部分，执行函数指令，通过返回地址，回到函数调用前）

```c++
//函数默认参数，要从后面的参数给出，中间不能有空缺，在函数声明中给出，不能在定义中给出
void show(int a,int b,int c = 3,int d = 4);
int main()
{
   show(1,2);
   return 0;
}
void show(int a,int b,int c,int d)
{
    std::cout << a << " " << b << " " << c << " " << d << std::endl;
}
```

**函数重载**：在同一作用域，函数名相同，参数列表不同（参数的个数或者参数的类型）、返回值可以不同

```c++
//void show(char *p) { printf("char *p\n"); }
//void show(char p[]) { printf("char p[]"); }		//不是重载，参数都是地址

void set(int a) { printf("int a"); }
void set(int &a) { printf("int &a"); }				//是重载，通过函数指针调用

void(*pfun1) (int a) = &set;
void(*pfun2) (int &a) = &set;

int main()
{
	//char *p = new char[6]();
	//show(p);
	//char cArr[9] = { 2,0,0,1 };
	//show(cArr);
	
	int a = 20;
	(*pfun1)(18);
	(*pfun2)(a);  
	return 0;

}
```

**函数重写（覆盖）**：父类与子类之间的虚函数，要求函数名参数列表（个数和类型），返回值都要相同

**函数隐藏**：是指派生类的函数屏蔽了与其同名的基类函数，注意只要同名函数，不管参数列表是否相同，基类函数都会被隐藏。

隐藏，指的是派生类类型的对象、指针、引用访问基类和派生类都有的同名函数时

**调用函数方法**

1. 函数名()
2. 函数指针调用

```c++
void Print(const char *str)
{
    std::cout << str << std::endl;
}
int main()
{
    void (*pfn) (const char *str);  //定义一个函数指针变量
    pfn = &Print;                   //给函数指针赋值
    (*pfn)("hello");                //调用函数指针
    return 0;
}
```

**如果给函数指针赋值的函数地址是类中的成员函数，那么这个函数应该是static类型的**

1. 类中的普通成员函数是有隐藏参数的
2. 调用类中的任意成员都需要用对象调用，如果普通成员函数可以给函数指针赋值，那么就可以不用通过对象调用，直接通过函数指针调用

```c++
class Entity
{
public:
    static void Print(const char *str)
    {
        std::cout << str << std::endl;
    }
};
int main()
{
    typedef void (*PFUN) (const char *str);
    //PFUN是一个函数指针类型
    PFUN pfn = &Entity::Print; 
    (*pfn)("hello");
    return 0;
}
```

**如果成员函数不能设置为静态的，就需要定义成员函数指针（将成员函数指针看作是类中的成员）**

```c++
//C++中提供新的操作符 ::* .* ->* 支持成员函数指针的操作
class Entity
{
public:
    void Print(const char *str)
    {
        std::cout << str << std::endl;
    }
};
int main()
{
    typedef void (Entity::*PFUN) (const char *str);  //定义一个成员函数指针的类型
    PFUN pfn = &Entity::Print;
    Entity e;
    (e.*pfn)("hello");
    Entity *pe = new Entity;
    (pe->*pfn)("hello");
    return 0;
}
```



**初始化列表**

​	初始化列表初始化内容：父类的构造（遗传自父亲的那份空间），成员（对象成员，const成员，成员初值）的构造或给初值

**const常量需要在初始化列表初始化** 

```c++
class Entity
{
public:
	const int m_a;
	int m_b;
public:
	Entity():m_a(6),m_b(6)	//初始化列表
	{
	}
public:
	void print()
	{
		std::cout << m_a << " " << m_b <<std::endl;
	}
};
```

**初始化列表执行顺序是变量定义顺序**

![image-20230311154158378](C++.assets/image-20230311154158378.png)

**一个类中包含另一个类的对象，这个类的初始化列表默认执行另一个类的无参构造，如果在构造函数中初始化对象，而不是在初始化列表初始化对象，会创建两次对象，造成性能浪费**

![image-20230311170600252](C++.assets/image-20230311170600252.png)

![image-20230312152850829](C++.assets/image-20230312152850829.png)





**定义子类对象，父类的成员在父类的构造中被初始化，先执行父类的构造（子类构造函数的初始化列表），再执行子类的构造，先执行子类的析构，在执行父类的析构**

**将父类中的析构函数置为虚函数，这样就先执行子类的析构函数，发现子类中有父类的成员（继承），然后在执行父类的析构函数**



## 虚函数

**父类的指针可以指向任意一个子类的对象**

+ 优点：统一类型，提高复用性
+ 缺点：只能使用父类的成员

```c++
class Entity
{
public:
	float X,Y;
	Entity()
	{
		X = 6;
		Y = 6;
	}
	void Move(float xa,float xb)
	{
		X+=xa;
		Y+=xb;
	}
};
class Player : public Entity
{
public:
	const char *Name;
};
int main()
{
	Entity *pe = new Player;			//创建一个堆区的对象
	std::cout << pe->X << std::endl;
	pe->Move(6,6);
	return 0;
}

```

**解决父类的指针只能使用父类成员的办法：父类的指针调用指向子类函数的一个函数指针**

```c++
class Entity
{
public:
    void Print()
    {
        std::cout << "Entity" << std::endl;
    }
};
class Player : public Entity
{
public:
    void Print()
    {
        std::cout << "Player" << std::endl;
    }

};
int main()
{
    typedef void (Entity::*PFUN) ();  //定义一个成员函数指针的类型
    PFUN pfn = (PFUN)&Player::Print;
    Entity *pe = new Player;
    (pe->*pfn)();
    return 0;
}
```



通过父类的指针调用实际的子类的成员函数，**虚函数的作用**主要是为了实现多态的机制。

**虚函数与普通函数的区别：**普通函数可以直接找到函数的地址来进行调用，虚函数需要通过虚指针和和虚函数列表来找到虚函数地址进行调用 （空指针可以直接调用普通函数而不能调用虚函数）  



**虚析构**

​	通过父类的指针完整删除一个子类的对象

```c++
class EntityBase
{
public:
	EntityBase()
	{
		std::cout << "EntityBase" << std::endl;
	}
	virtual ~EntityBase()
	{
		std::cout << "~EntityBase" << std::endl;
	}
};
class Entity : public EntityBase
{
public:
	Entity()
	{
		std::cout << "Entity" << std::endl;
	}
	~Entity()
	{
		std::cout << "~Entity" << std::endl;
	}
};
int main()
{
	{
		EntityBase *p = new Entity;
		delete p;
	}
	system("pause");
	return 0;
}
```

## **多态**

​	用父类型的指针指向其子类的实例，然后通过父类的指针调用实际子类的成员函数。这种技术可以让父类的指针有“多种形态”，这是一种泛型技术。

**多态的条件：**

+ 存在继承关系
+ 父类的指针指向子类对象
+ 父类中存在虚函数，子类重写了虚函数

重写：子类中存在和父类一摸一样的虚函数

**多态的优缺点：**

+ 优点：调高复用性和扩展性
+ 缺点：空间（虚指针，虚表）、效率（调用）、安全性（私有函数不能定义为虚函数，在类外可以调用）

**虚函数实现多态原理**

1. 函数指针
2. 需要维护一个由函数指针组成的虚函数列表（vtable），还需要记录使用那个列表的指针（vfptr）
3. 当调用一个虚函数，通过vfptr找到对应的vtable，调用表里的函数指针，如果在子类中重写了，表中装的就是子类的函数（覆盖），调用的也就是子类
4. vtable每个类包含一个编译期存在，vfptr是在创建对象的时候存在，每个对象有一个，在构造函数里初始化，指向对应的虚函数列表

**动态多态的具体实现过程：**

1. 父类要有虚函数，父类就有一个虚函数表

2. 子类继承父类，子类就有一个虚函数表

   子类重写虚函数，就会覆盖虚函数表里面虚函数地址

3. 定义子类对象，子类对象的前4个字节就会有一个虚指针，虚指针指向虚函数表

   父类指针指向子类对象

4. 父类指针调用虚函数，就会根据子类对象的前四个字节，得到虚指针，进而得到虚函数表的入口地址，通过遍历虚函数表，找到对应的虚函数并调用，从而完成多态

在定义对象的内存空间的前面分配__vfptr,在执行构造函数的初始化参数列表进行了初始化，指向了虚函数列表

**虚函数多态，可以利用引用实现多态**

![image-20220222214846555](C++.assets/image-20220222214846555.png)

```c++
class EntityBase
{
public:
	virtual std::string GetName() { return "EntityBase"; };                     
};

class Entity : public EntityBase
{
private:
	std::string m_Name;
public:
	Entity(const std::string& name)
		:m_Name(name) {}
	std::string GetName() { return m_Name; }
};

void PrintName(EntityBase *eb)
{
	std::cout << eb->GetName() << std::endl;
}

int main()
{
	EntityBase *eb = new EntityBase;
	PrintName(eb);
    
	EntityBase *e = new Entity("ZH");
	PrintName(e);
    
	system("pause");
	return 0;
}
```



```c++
class Father
{
public:
	virtual void AA()
	{
		std::cout << "Father::AA" << std::endl;
	}
	virtual void BB()
	{
		std::cout << "Father::BB" << std::endl;
	}
	void CC()
	{
		std::cout << "Father::CC" << std::endl;
	}
};
class Son : public Father
{
public:
	virtual void AA()
	{
		std::cout << "Son::AA" << std::endl;
	}
	void CC()
	{
		std::cout << "Son::CC" << std::endl;
	}
	virtual void DD()
	{
		std::cout << "Son::DD" << std::endl;
	}
};
int main()
{
	Father *p = new Son;	
	typedef void (*PFUN) ();
	PFUN pfn1 = (PFUN)*((int*)*(int*)p+0);//函数指针不能偏移
	PFUN pfn2 = (PFUN)*((int*)*(int*)p+1);
	PFUN pfn3 = (PFUN)*((int*)*(int*)p+2);
	PFUN pfn4 = (PFUN)*((int*)*(int*)p+3);//第一次强转(int*)是取对象前四个字节的内容也就是vfptr，第二次强转是取函数指针											   数组的前四个字节的内容
	(*pfn1)();
	system("pause");
	return 0;
}
```

![image-20220222220953320](C++.assets/image-20220222220953320.png)



**虚继承**

​		虚继承是解决C++多重继承问题的一种手段，从不同途径继承来的同一基类，会在子类中存在多份拷贝。这将存在两个问题：其一，浪费存储空间；第二，存在二义性

​		虚继承的引入主要是为了解决多继承环境下有歧义的层次组合问题（通常被称为“钻石问题”）

```c++
class Base
{
public:
    Base(int n) : value(n)
    { 
        std::cout << "Base(" << n << ")"<< std::endl; // 输出传入值：Base(N)
    }
    Base() : value(0)
    { 
        std::cout << "Base()"<< std::endl; // 没有传入值：Base()
    }
    ~Base() { std::cout << "~Base()"<< std::endl; }
    int value;
};
class One : public Base
{
public:
    One() : Base(1) 
    { 
        std::cout << "One()"<< std::endl; 
    }
    ~One() { std::cout << "~One()"<< std::endl; }
};
class Two : public Base
{
public:
    Two() : Base(2)
    { 
        std::cout << "Two()"<< std::endl; 
    }
    ~Two() { std::cout << "~Two()"<< std::endl; }
};
class Leaf : public One, public Two
{
public:
    Leaf() : { std::cout << "Leaf()"<< std::endl; }
    ~Leaf() { std::cout << "~Leaf()"<< std::endl; }
};
```

![image-20221101135841864](C++.assets/image-20221101135841864.png)![image-20221101140044179](C++.assets/image-20221101140044179.png)

在这个实现中，Leaf实例持有两个`Base`类的拷贝：第一个来自于`One`，第二个来自于`Two`。这样的实现使得下面语句：

```c++
Leaf lf;
lf.value = 0; // 编译错误！
```

通常，我们会尽量避免一个`Leaf`对象持有多个`Base`类。这可以通过使用虚继承实现：我们可以在继承的子类添加`virtual`关键字：`class One : public `**virtual**` Base…`以及`class Two : public `**virtual**` Base…`。使用`virtual`关键字，`Leaf`类仅会调用一次`Base`类的构造函数，因而也就只构建了一个`Base`。`Leaf`内部只持有一个`Base`子对象（该子对象也会被`One`和`Two`“共享”）。这就是我们所需要的。

那么问题来了，“编译器怎么知道该给`Base`的构造函数传哪个参数？”的确，我们有两个选择：`One`的构造函数调用了`Base(1)`，`Two`的构造函数调用了`Base(2)`。那么，我们究竟该选哪个？答案很明显：哪个都不选。`Leaf`的构造函数**直接**调用时，编译器选择的是`Base()`的默认构造函数，而不是`Base(`**1**`)`或者`Base(`**2**`)`。 

编译器会隐式在`Leaf`初始化列表中添加`Base()`的调用，并忽略其它`Base()`构造函数的调用。因此，初始化列表类似这样：

```c++
class Leaf : public Base(), public One, public Two
{
    ...
}
//当然，我们也可以给Leaf的初始化列表显式添加Base(...)语句，例如，我们需要给构造函数传入参数 3，那么就可以这么写：
//class Leaf : public Base(3), public One, public Two
{
    ...
}
```

**虚继承应用**

+ 最终类

所谓“最终类”，是一种能够在堆上或者栈上创建实例，但是不能被继承的类。换句话说，下面的代码是合法的：

```c++
Final fd;
Final *pd = new Final();
//但是，下面的代码则会给出一个编译错误：
class Derived : public Final{};

//在 C++ 标准引入final关键字之前的很长一段时间，最终类问题都是通过虚继承解决的。例如More C++ Idioms/Final Class这里所阐述的。其解决方案如下所示：

class Seal
{
    friend class Final;
    Seal() {}
};
class Final : public virtual Seal
{
public:
    Final() {}
};
//继承Final会引发一个编译错误“不能访问Seal类的私有成员”：

class Derived : public Final
{
public:
    Derived() {} // 编译错误：
                 // cannot access private member of class Seal
};
```

这个技巧的关键是虚继承：`Derived`构造函数必须直接调用`Seal`的构造函数，而不能通过`Final`的构造函数间接调用。但是，这又是不允许的：`Seal`类只有私有构造函数，而`Derived`类又不是像`Final`那样是`Seal`的友元。`Final`类本身允许调用`Seal`的私有构造函数，因为它是`Seal`的友元。这就是为什么`Final`可以在栈上或者堆上创建对象。

这个解决方案建立在虚继承的基础之上。如果移除继承声明`class Final : public `**virtual**` Seal`中的`virtual`关键字，`Derived`类就可以通过`Final`类间接调用`Seal`的构造函数（因为后者是`Seal`的友元），这个魔法就消失了。

**类型转换函数特征：**

1. 类型转换函数定义在源文件中
2. 须由operator修饰，函数名称是目标类型名或目标类名
3. 函数没有参数，没有返回值，但是有return语句，return目标类型数据或调用目标类的构造函数

**类型转换函数主要有两类：**

+ 对象向基本数据类型转换
+ 对象向不同类的对象的转换

## 重载操作符

​	必须要对象参数，而且需要返回值，这个返回值急需的和其他符号结合

**重载操作符方式有两种：**

1. 类内重载，对象一定要在符号的左边

```c++
int operator=(int num);
int CPerson::operator=(int num)
{
	m_Age = num;
}
CPerson ps;
ps = 100;
std::cout<<ps.m_Age<<std::endl;

int  CPerson::operator+(CPerson &ps)	
{
	return m_Age+ps.m_Age;
}
CPerson ps2;
ps2 = ps1+ps;					//对象在重载操作符的左边，在类内重载，函数参数只关系重载操作符右边变量的类型
std::cout<<ps2.m_Age<<std::endl;
```

```c++
CPerson &operator=(const CPerson &ps)	//类默认的operator= 浅拷贝
{
	this->m_Age = ps.m_Age;
	return *this;
}
```

```c++
CPerson &operator=(const CPerson &ps)	//operator= 深拷贝
{
	//删除原有的空间
	delete this->m_Age;
	this->m_Age = new int;
	*(this->m_Age) = *(ps.m_Age);
	return *this;
}
```

2. 类外重载，对象要在符号的右边（双目运算符）

```c++
int operator+(int num,CPerson &ps)	//参数的顺序就是重载操作符左右边的顺序
{
	return num+ps.m_Age;
}
ps2 = 100+ps1;
std::cout<<ps2.m_Age<<std::endl;
```

```c++
std::istream& operator>>(std::istream &is,CPerson& ps)
{ 
	is >> ps.m_Age;
	return is;
}
std::ostream& operator<<(std::ostream &os,CPerson &ps)
{
	os<<ps.m_Age;
	return os;
}
std::cin>>ps1;
std::cout<<ps1;
```

```c++
int CPerson::operator++(int a)	//右加加，参数的作用是区分左加加和右加加
{
	int flag = m_Age;
	m_Age+=1;
	return flag;
}
int CPerson::operator++()		//左加加
{
	m_Age+=1;
	return m_Age;
}
```

![image-20230316122731128](C++.assets/image-20230316122731128.png)

![image-20230316132201583](C++.assets/image-20230316132201583.png)

## 拷贝构造函数

​	构造函数的参数是当前这个类的一个const类型的引用，默认是浅拷贝（复制一个对象，把类中的成员变量复制一份，如果成员变量中有指针，会使两个指针指向同一块空间，对象生命周期结束时会调用两次析构函数，释放两次，引发错误），避免浅拷贝的方法是参数为指针或者引用，或者深拷贝。

```c++
class CPerson
{
public:
	int *m_Age;
public:
	CPerson(){
		m_Age = new int(100);
	}
	CPerson(const CPerson &ps){    //默认的是浅拷贝		
		m_Age = ps.m_Age;		   //把ps的m_Age赋值给ps1的m_Age 
	}
	CPerson(const CPerson *ps){    // 深拷贝
		this->m_Age = new int;
		*(this->m_Age) = *(ps->m_Age);
	}
	~CPerson(){
		delete m_Age;
		m_Age = 0;
	}
};
int main()
{
	CPerson ps;
	CPerson ps1(ps);
	system("pause");
	return 0;
}
```

```c++
void QQ(CPerson ps1){}	//CPerson ps1相当于CPerson ps1(ps)改成CPerson &ps1可以避免浅拷贝
CPerson ps;
QQ(ps);
```

**何时调用构造函数**

1. 类名 新对象 （已存在的对象）
2. 类名 新对象 = 已存在的对象
3. 类名 *新对象 = new 类名(已存在的对象)
3. 函数参数传递时，将实参传递给形参
3. 函数返回一个对象时



**隐式转换**

![image-20230317003336241](C++.assets/image-20230317003336241.png)

![image-20230317141847808](C++.assets/image-20230317141847808.png)



## 函数模板

​	模板函数并不算是一个真正的函数，可以把它看作是一个函数生成器。一个模板并不会被自发地编译成object文件中的机器码，只有当模板被调用时编译器才会对它展开、实例化，被编译成机器码。

![image-20230317204400823](C++.assets/image-20230317204400823.png)

**template：**定义模板的关键字

**typename：**定义模板类型的关键字

**<>：**模板的参数列表 

如果函数的声明和定义分开，那么在声明和定义处都需要加上模板，如果模板存在默认值，那么只在函数声明时指定即可。

**注意事项：**如果函数的声明和定义分开，因为源文件是单独编译的，如果模板函数定义所在的源文件中并没有使用模板函数，那么模板函数就不会被实例化，在主函数就会找不到模板函数的定义。

**模板实例化：**编译器由函数模板自动生成模板函数的过程叫模板的实例化

**隐式实例化：**编译器能够根据实参自动推导出模板类型，并生成对应的类型的函数

**优先级：手动显示指定 > 通过实参自动推导 > 模板类型默认值**

```c++
#include<iostream>

template<typename X,typename Y=float,typename Z>	//默认模板参数
X add(Y y, Z z) {
	std::cout << typeid(y).name() << std::endl;
	std::cout << typeid(z).name() << std::endl;
	X x = 666;
	std::cout << typeid(x).name() << std::endl;
	return y + z;
}

int main()
{
	int a = 1;
	double b = 6.6;
	/*std::cout << add<int, double, long long>(a, b) << std::endl;*/	//显示地指定模板参数类型
	/*std::cout << add(a, b) << std::endl;*/
	std::cout << add<long long>(a, b) << std::endl;
	return 0;
} 
```

​	不同于函数参数的默认值，模板参数默认值指定的顺序可以是任意的，因为参数类型由多种因素决定，但在显示指定模板类型时必须从左向右依次指定，不能有间断

​	编译器能够自动推导出来的模板参数放于最后，有默认值的模板参数放于中间，无默认值则放于前面。（不是必须，只是可以减少显示地指定）

**显示实例化：**由编程者决定生成指定模板类型的函数，不管其是否被使用到。（即使不调用模板，也可以生成实例化的函数）

```c++
template <typename T>
T a(T t1, T t2)
{
	return t1 > t2 ? t1 : t2;
}

template int a(int, int);
```

​	模板函数具有一定的局限性，同一功能并不是所有类型的操作都是一致的，比如 int类型 和 struct 类型的swap功能。所以有时需要为特定的类型指定特定的操作，而不是千篇一律使用通用的模板函数，此时可以**显式具体化**。

**显示具体化（显示专用化、特化）：**与显式实例化不同，告诉编译器不要使用模板函数来生成对应模板类型的函数定义， 而使用模板类型重新生成新的函数定义，所以显式具体化一定要有函数的定义，在函数前加上 template<>，其必须依赖于模板函数才能成立。

​	编译器在实例化模板的过程中，会优先选择特化的模板

​	使用函数重载可以实现函数模板特化的功能。如果使用重载函数，不管是否发生实际的函数调用，都会在目标文件中生成该函数的二进制代码，如果使用模板特化，除非发生函数调用，否则不会在目标文件中包含特化的模板函数的二进制代码。如果使用普通重载函数，那么在分离编译模式下，需要在各个源文件中包含重载函数的申明，否则在某些源文件中就会使用模板函数，而不是重载函数。

​	虚函数依赖的是运行时刷新虚表，查询虚表，才能达到多态的效果，模板特化在编译时就完成了分支的选择，不占用运行时的资源，所以它的性能更优

**全特化：**<>中没有任何模板参数，表示所有的模板参数都被特别指定了。

```c++
#include<stdio.h>
#include<string.h>
#include<type_traits>

template <typename T>
T MYMAX(T a, T b)
{
	static_assert(std::is_integral<T>::value || std::is_floating_point<T>::value,
		"T must be integral or floating point");
	return a > b ? a : b;
}
template<> double MYMAX(double a, double b)		//显示具体化，针对某一具体的类型，有专用的实现
{
	return a > b ? b : a;
}
template <>
const char* MYMAX(const char* a,const char* b){
	if(strlen(a) > strlen(b))	return a;
	return b;
}
int main()
{
	printf("%d\n", MYMAX(6, 7));    
	printf("%lf\n", MYMAX(1.2, 3.4));
	printf("%s\n", MYMAX("zh", "zhong"));		//比较内存地址
	return 0;
}
```

**static_assert：**C++11新语法，在编译时做必要的检查，如果检查不通过，直接打断编译，并且给出错误信息

**同类型的显示专用化模板函数和全局函数可以同时存在**

**同类型的显示实例化模板函数和显示专用化模板函数不能同时存在**

**优先级：**全局函数>显示专用化>普通模板函数

```c++
template<typename T>
void show(T t) {
	std::cout << "普通函数" << std::endl;
}

template void show(int n);				//显示实例化

//template void show(char c);
template<> void show(char c) {
	std::cout << "模板特化" << std::endl;
}
void show(char c) {
	std::cout << "全局函数" << std::endl;
}

int main()
{
	show(1);
	show('a');			//模板特化
	show<>('b');		//不调用全局函数，调用模板特化
	show(1.2);
	std::cin.get();
	return 0;
}
```



## 类模板

​	类模板不能自动推导模板参数类型，只能显示地指定，所以默认的模板参数只能从右向左不能间断

```c++
template<typename T,typename K=int>		//类模板不能自动推导模板参数类型，只能显示地指定，
										//所以默认的模板参数只能从右向左不能间断
class Entity {
public:
	T m_a;
	K m_b;
	Entity(){
		m_a = 6;
		std::cout << typeid(m_a).name() << std::endl;
	}
	Entity(T &t) {
		m_a = t;
		std::cout << typeid(m_a).name() << std::endl;
	}
	Entity(const T &t,const K &k) {
		m_a = t;
		m_b = k;
		std::cout << typeid(m_a).name() << std::endl;
		std::cout << typeid(m_b).name() << std::endl;
	}
public:
	void show();

	template<typename P>
	void play(T t);

	template<typename G>
	void get(G g);
};

template<typename T, typename K>
void Entity<T,K>::show()
{
	std::cout << "show()" << std::endl;
	std::cout << typeid(T).name() << std::endl;		//T、K的类型由<double,long>决定
	std::cout << typeid(K).name() << std::endl;
}

template<typename T,typename K>
template<typename P>
void Entity<T, K>::play(T t)
{
	P p;
	std::cout << typeid(p).name() << std::endl;
	std::cout << typeid(t).name() << std::endl;
}

template<typename T,typename K>
template<typename G>
void Entity<T, K>::get(G g)
{
	std::cout << typeid(g).name() << std::endl;
}

int main()
{
	Entity<double> e;
	std::cout << e.m_a << std::endl;

	char c = 'a';
	Entity<char> e1(c);
	std::cout << e1.m_a << std::endl;

	Entity<long, double> e2(6, 6.06);	//<>指定类模板类型
	std::cout << e2.m_a << std::endl;
	std::cout << e2.m_b << std::endl;

	e2.show();
	e2.play<char>(666);					//<>指定函数模板类型
	e2.get("ZH");						//函数模板自动推导

	system("pause");
	return 0;
}
```

```c++
template<typename T>
class CA
{
public:
	T m_a;
	CA(){
		m_a = 65;
	}
	CA(const T &a) {
		m_a = a;
	}
};

template<typename K>
class CB
{
public:
	CA<K> a;
	CB() {}
	CB(CA<K> &a) {
		this->a = a;
	}

};

template<typename C>
class CC
{
public:
	C c;
};
int main()
{
	CA<double> a(20);
	CB<double> b(a);
	std::cout << b.a.m_a << std::endl;

	CB<char> b1;
	std::cout << b1.a.m_a << std::endl;

	CC<CA<double>> c;
	std::cout << c.c.m_a << std::endl;
	system("pause");
	return 0;

}
```



```c++
template <typename E>
class Entity
{
public:
    E m_mem;
public:
    Entity(E e){
        m_mem = e;
    }
};
int main()
{
    Entity<int> e(606);
    std::cout<< e.m_mem << std::endl;
    return 0;
    
};
//==============================================

template <typename E>
class Entity
{
public:
    E m_mem;
public:
    Entity(E e){
        m_mem = e;
    }
};
template<typename EE>
class EEntity
{
public:
    EE m_mem;
}
int main()
{
    EEntity<int> EE();
    Entity<EEntity<int>> e(EE);
    return 0;
    
};

```

**偏特化**

```c++
enum class DEVICE{
    CPU,
    GPU
};

template<DEVICE dev,typename IN_T1,typename IN_T2>
struct CExample{
    void operator()(IN_T1 a,IN_T2 b);
};

template<typename IN_T1,typename IN_T2> 
struct CExample<DEVICE::CPU, IN_T1, IN_T2> //在类名后面指定好实例化时的模板参数
{
    void operator()(IN_T1 a,IN_T2 b){
        std::cout << "CPU:" << a << " " << b << std::endl;
    }
};

template<typename IN_T1,typename IN_T2> 
struct CExample<DEVICE::GPU, IN_T1, IN_T2>
{
    void operator()(IN_T1 a,IN_T2 b){
        std::cout << "GPU:" << a << " " << b << std::endl;
    }
};

int main()
{
    CExample<DEVICE::GPU,int,int>() (6,6);
    CExample<DEVICE::CPU,float,float>() (6.6,6.6);
	return 0;
}
```

**模板类型推导**

+ 参数的类型为指针或者引用

  ```c++
  int n = 6;
  const int nn = n;
  const int& nnn = n;
  
  template<typename T>
  void f(T& param);
  f(n);	//T is int,param's type is int&
  f(nn);	//T is const int,param's type is const int&
  f(nnn);	//T is const int&,param's type is const int&
  
  template <typename T>
  void f(const T& param);
  f(n);	//T is int,param's type is const int&
  f(nn);	//T is int,param's type is const int&
  f(nnn); //T is int,param's type is const int&
  
  template <typename T>
  void f(T* param);
  const int *p = &n;
  f(&n);	//T is int,param's type is int*
  f(p);	//T is const int,param's type is const int*
  ```

+ 参数的类型是万能引用

  ```c++
  int n = 6;
  const int nn = n;
  const int& nnn = n;
  
  template <typename T>
  void f(T&& param);
  f(n);	//n is lvalue,T is int&,param's type is also int&
  f(nn);	//nn is lvalue,T is const int&,param's type is also const int&
  f(nnn);	//nnn is lvalue,T is const int&,param's type is also const int&
  f(666);	//666 is rvalue,T is int,param's type is int&&
  ```

  

+ 参数的类型既不是指针也不是引用通常是值传递

  ```c++
  int n = 6;
  const int nn = n;
  const int& nnn = n;
  
  //值传递会去掉const
  template <typename T>
  void f(T param);
  f(n);	//T's and param's types are both int
  f(nn);	//T's and param's types are both int
  f(nnn);	//T's and param's types are both int
  
  const char* const ptr = "zhong";
  f(ptr);	//T's and param's types are both const char*
  
  const char name[] = "zhong";	//name's type is const char [6];
  const char *ptrToName = name;	//array decays to pointer
  f(name);	//T deduced as const char*
  
  template <typename T>
  void f(T& param);
  f(name);	//T is deduced to be const char[6] 
  
  //return size of an array as a compile-time constant
  template<typename T,std::size_t N>
  constexpr std::size_t arraySize(T (&)[N]) noexcept{
      return N;
  }
  int KeyVals[] = {1,3,5,7,9};
  int mappedvals[arraySize(KeyVals)];
  ```

  

## 变参模板

​	可以让函数或者类的方法有接受任意个参数的能力

**typename...：**修饰变参模板的模板参数

**Args...：**告诉编译器函数的参数是可变的

```c++
#include<iostream>

void myprint() {}

template<typename T,typename... Args>
void myprint(T firstArg, Args... args) {
	std::cout << firstArg << std::endl;
	myprint(args...);
}

int main()
{
	myprint(2001, 6, 6, "zh");
	return 0;
}
```

**Args、args 解包(pack expansion)：**第一次调用myprint时，第一个参数firstArg被推导成了int型的，Args...被推导成了int型的E1、int型的E2、const char *型的E3.剩下的情况一样，直到最后myprint()的参数为空，函数结束，这也就是为什么在上面定义了myprint()的原因



**sizeof...**

​	帮助我们在编译的时候就知道函数被调用时到底传了几个参数

```c++
template<typename... Args>
void func(Args... args) {
	std::cout << "sizeof...(Args)" << sizeof...(Args) << std::endl;
	std::cout << "sizeof...(args)" << sizeof...(args) << std::endl;
	std::cout << "---------------" << std::endl;
}
int main()
{
    func(2001,6,6,"zh");
    func(2001, 6, 6);
	func(2001, 6);
	func(2001);
	func();
	return 0;
}
```



## 类型转换

​	强调类型安全，收到编译时的检查，减少在尝试强制类型转换时可能会意外犯的错误，比如类型不兼容，还可以帮助我们搜索在哪使用了类型转换

隐式转换

显示转换

**static_cast**：编译时确定的，相关内容的转换，（向下转换，没有检查，会出现问题）

**dynamic_cast**：只适用于继承链上的两个类之间转换，向上转换等同于static_cast

向下转换需要进行校验，向下转换安全（即父类指针要转换为子类指针，恰巧这个父类指针指向这个子类对象），向下转换不安全（即父类指针要转换为子类指针，恰巧这个父类指针不指向这个子类对象），返回为空，向下转换父类中要包含虚函数

特殊情况：即父类指针要转换为子类指针，恰巧这个父类指针指向这个子类的子类对象

**reinterpret_cast**：重解释类型转换，是C++里的强制类型转换。

**const_cast**：去常转换 const->非const volatile->非volatile

**为什么调用dynamic_cast？**

父类指针指向子类对象，可以正确的调用子类特有方法

```c++
class Entity
{
	virtual void PrintName() {}
};
class Player : public Entity{};
class Enemy : public Entity{};
int main()
{
	Player *player = new Player();
	Entity *e = player;						//隐式转换
	Entity *e1 = new Enemy();
	Entity *e2 = new Entity;
	Player *p = dynamic_cast<Player*>(e);   
	Player *p1 = dynamic_cast<Player*>(e1);	//NULL
	Player *p2 = dynamic_cast<Player*>(e2);	//NULL
	system("pause");
	return 0;
}
```

## ( ) 与 { }

```c++
class Widget{	//C++11开始可以这样进行初始化
private:
    int x{0};   //fine
    int y = 0;  //fine
    int z(0);   //error
};
Widget w1;  //call default constructor
Widget w2 = w1; //not an assignment;calls copy ctor
w1 = w2;    //an assignment;calls copy operator=

Widget w3(666);   //call Widget ctor with argument
Widget w4();      //这有可能是一个无参构造也有可能是一个返回值为Widget的函数，
                  //C++规定可以解析成声明语句都优先解析成一个声明语句,所以这是一个函数的声明
Widget w5{};      //calls Widget ctor with no args

std::atomic<int> ai1{0};    //fine
std::atomic<int> ai2(0);    //fine
std::atomic<int> ai3 = 0;   //error

double x,y,z;
int sum{x+y+z}; //统一初始化不允许隐式窄化

std::vector<int> v1(6,6);   //ctor:creat 6-element std::vector,
                            //all elements have value of 6;
std::vector<int> v2{6,6};   //use std::initializer_list ctor:
                            //create 2-element std::vector,
                            //element values are 6 and 6;

template<typename T,typename... Ts>
void doSomeWork(Ts&&... params){
    // T localobject(std::forward<Ts>(params)...);
    // T localobject{std::forward<Ts>(params)...};
}
std::vector<int> v;
doSomeWork<std::vector<int>>(10,20);
```

![image-20230325202348408](C++.assets/image-20230325202348408.png)



```c++
class Widget{
public:
    Widget(int i,bool b){
        std::cout << "first" << std::endl;
    }
    Widget(int i,double d){
        std::cout << "second" << std::endl;
    }
    Widget(std::initializer_list<long double> il);
    operator float();
private:
};

int main()
{
    Widget w1(10,true); //calls first ctor
    Widget w2{10,true}; //calls third ctor 用花括号初始化器列表列表初始化一个对象，
    					//其中对应构造函数接受一个std::initializer_list参数
    Widget w3(10,5.0);  //calls second ctor
    Widget w4{10,5.0};  //calls third ctor {}中的类型可以转换成或等于<>中的类型就可以
    Widget w5(w4);  	//uses parens,calls copy ctor
    Widget w6{w4};  	//uses braces,calls std::initializer_list ctor
                   		//w4 converts to float,and float converts to long double
    Widget w7(std::move(w4));   //uses parens,calls move ctor
    Widget w8(std::move(w4));   //uses braces,calls std::initializer_list ctor,for same reason as w6
    return 0;
}
```

```c++
class Widget{
public:
    Widget(int i,bool b);
    Widget(int i,double d);
    Widget(std::initializer_list<bool> il);
private:
};

int main()
{
    Widget w{10,5.0};   //error! requires narrowing conversions,统一初始化不允许窄化
    return 0;
}
//========================================================================================================
class Widget{
public:
    Widget(int i,bool b);
    Widget(int i,double d);
    Widget(std::initializer_list<std::string> il);
private:
};

int main()
{
    Widget w1(10,true);   //uses parens calls first ctor
    Widget w2{10,true};   //uses braces calls first ctor 参数不能转换
    Widget w3(10,5.0);    //uses parens calls second ctor
    Widget w4{10,5.0};    //uses braces calls second ctor
    return 0;
}
```

```c++
class Widget{
public:
    Widget();
    Widget(std::initializer_list<int> il);
private:
};

int main()
{
    Widget w1;      //calls default ctor
    Widget w2{};    //calls default ctor
    Widget w3();    //declares a function
    Widget w4({});  //calls std::initailizer_list ctor with empty list
    Widget w5{{}};  //ditto
    return 0;
}
```



# C++11

```c++
//范围for
int arr[3] = {0,1,2};
for(auto val:arr){
    std::cout << val << std::endl;
}
```

**auto**

​	自动推导出数据的类型

**优点：**

1. 当数据类型过长或者迭代器名字过长时可以用auto
1. 当使用lamda时可以用auto
2. 当改变数据类型不需要对数据类型做出改变

**缺点：**

1. 减低代码可读性，不知道数据的类型
2. 会破坏依赖于特定类型的代码  

```c++
char *GetName()
{
	return "Skr";
}
int main()
{
	std::string str = GetName();
	//auto str1 = GetName();
	int n = str.size();
	std::cout << n << std::endl;
	std::cin.get();
	return 0;
}
```

## **右值引用**

​	解决了移动语义、完美转发

**左值**：有内存地址的表达式，可以出现在等号的左边、右边

**右值：**没有内存地址的表达式，只能出现在等号的右边

**右值的情况：**

1. 常量字面量（没有地址）如10、"hello"
2. 函数调用返回的值或对象（返回的是左值引用除外）
3. 构造的无名对象

对以上三种情况取地址都会出错

为了实现**移动语意，**C++ 增加了与拷贝构造函数（copy constructor）和拷贝赋值操作符（copy assignment operator）对应的移动构造函数（move constructor）和移动赋值操作符（move assignment operator），通过函数重载机制来确定应该调用拷贝语意还是移动语意（参数是左值引用就调用拷贝语意；参数是右值引用就调用移动语意）

右值引用和std::move被广泛用于在STL和自定义类中**实现移动语义，避免拷贝，从而提升程序性能**

在STL的很多容器中，都实现了以右值引用为参数的移动构造函数和移动赋值重载函数，或者其他函数，最常见的如std::vector的push_back和emplace_back。参数为左值引用意味着拷贝，为右值引用意味着移动

**移动构造：**形参是一个右值引用，把被移动对象的成员指针置为空（以避免移动过来的内存被析构），减少深拷贝操作	

**std::move:**强制执行表达式为右值，

```c++
#include<iostream>
#include<string>
class CMyString
{
public:
	char *m_pBuffer;
	int m_iLen;
	CMyString(const char *pString)
	{
		m_iLen = strlen(pString)+1;
		m_pBuffer = new char[m_iLen];
	}
	~CMyString()
	{
		m_iLen = 0;
		if(m_pBuffer != nullptr)
			delete[] m_pBuffer;
	}
	CMyString(const CMyString& other)
	{										//深拷贝
		this->m_iLen = other.m_iLen;
		this->m_pBuffer = new char[m_iLen];
		strcpy_s(m_pBuffer, m_iLen, other.m_pBuffer);
	}
	CMyString(CMyString&& other)			//	移动构造，形参是一个右值引用
	{
		//this->m_iLen = other.m_iLen;
		//this->m_pBuffer = other.m_pBuffer;
		std::cout << "移动构造" << std::endl;
	} 
	void operator=(CMyString && other) {
		std::cout << "重载=" << std::endl;
	}
};
template<class T>
void swap(T& a, T& b)
{
	T tmp(std::move(a));			//执行的是深拷贝，如果是移动拷贝效率更高
	a = std::move(b);
	b = std::move(tmp);
}
int main()
{
	//CMyString str(CMyString("hello world"));//被拷贝的对象是一个无名对象，生命周期只有这一行，开辟的空间可以直接给str,												叫偷资源
	CMyString str1("hello");
	CMyString str2("world");
	swap<CMyString>(str1, str2);
	return 0;
}
```

**move**：本质上是做了一个类型转换，remove_reference将我们传入的参数去掉&，最后加上&&

**注意事项：**返回局部变量时不要使用move

```c++
template<class _Ty>
	_NODISCARD constexpr remove_reference_t<_Ty>&&
		move(_Ty&& _Arg) noexcept
	{	// forward _Arg as movable
	return (static_cast<remove_reference_t<_Ty>&&>(_Arg));
	}

template<class _Ty>
	struct remove_reference
	{	// remove reference
	using type = _Ty;
	};

template<class _Ty>
	struct remove_reference<_Ty&>
	{	// remove reference
	using type = _Ty;
	};

template<class _Ty>
	struct remove_reference<_Ty&&>
	{	// remove rvalue reference
	using type = _Ty;
	};
```

## **lambda**

​	只要有函数指针就可以使用lamda

​	**函数作为一个参数出现时，需要在函数外写函数的实现，以前无法在函数内实现另一个函数，lamda表达式，可以定义函数的对象，可以在函数内部实现函数定义**

​	**当函数作为一个参数出现时，可以直接写函数的定义**

[ ]（）mutable ->type {  }

1. 捕获列表：捕获（lamda表达式）外面的局部变量，以及this指针（静态和全局的不用捕获，直接能用）

​	[=]	捕获外面的非静态局部的全部变量的副本，值传递，以及this指针

​	[&]	捕获外面的非静态局部的全部变量的副本，引用传递，以及this指针

​	[val]  捕获val变量，值传递

​	[&val]  捕获val变量，引用传递

​	[this] 	捕获this指针

​	[=,&val]	捕获val是引用，其他变量的捕获是指传递

​	[&,val]	捕获val是指传递，其他事引用传

2. 参数列表：与函数的参数列表是一样的，不过可以省略，但是如果有mutable修饰时，列表不能省略，是少要有（）
3. 是否为常函数：限制的是值传递，如果没有就为常函数
4. 返回值类型
5. 函数体：与函数写法一致

```c++
void ForEach(const std::vector<int>& values,void(*func)(int))
{
    for(int value:values)
        func(value);
}

int main()
{
    std::vector<int> values = {1,2,3,4,5};
    ForEach(values,[](int value){std::cout << "value:"<< value << std::endl;});
    return 0;
}
```

**为什么想要用lambda**

**std::function是函数模板类**

**find_if：用来在某种迭代器中找到值，返回第一个符合条件的迭代器**

```c++
void ForEach(const std::vector<int>& values,const std::function<void(int)> &func)		//func是一个函数对象
{
    for(int value:values)
        func(value);
}
int main()                                                    
{
    std::vector<int> values = {1,2,3,4,5};
    auto ite =  std::find_if(values.begin(),values.end(),[](int value){return value > 1; });
    std::cout << *ite << std::endl;
    int a = 5;
    auto lambda = [&a](int value){a = 6;std::cout << "value:" << a << std::endl;};//当使用值传递或者引用传递时不																					能用原始函数指针来调用lambda
    ForEach(values,lambda);
    return 0;
}
```

**野指针**

​	指针指向的位置是不正确的

产生原因：释放内存后指针没有置为NULL

危害：会出现非法访问的错误

避免方法：

+ 初始化置NULL	
+ 申请内存判断是不是为NULL
+ 指针释放后置NULL
+ 使用智能指针

## **智能指针**

​	万物皆可对象，指针封装成类，构造，析构回收指针所指向的空间

**智能指针是RAII(Resource Acquisition Is Initialization，资源获取即初始化)机制对普通指针进行的一层封装。这样使得智能指针的行为动作像一个指针，本质上却是一个对象**

作用：防止忘记delete释放内存，造成内存泄漏

+ **auto_ptr(c++98之后)：**使用交出的太随意了，交出后，原指针为空，失效

+ **unique_ptr：**作用域指针，超出作用域会被销毁，独享被管理对象指针所有权，不能拷贝和赋值，没有拷贝构造函数（转移指针控制权std::move()，可以通过移动构造函数，如果只是单纯想使用，函数传参（不可以直接传，会触发拷贝），可以用get来获取指针），如果其中一个失效，那么就会释放指向的内存。

  ```c++
  void f1() {
      unique_ptr<int> p(new int(5));
      unique_ptr<int> p2 = std::move(p);
      //error，此时p指针为空: cout<<*p<<endl; 
      cout<<*p2<<endl;
  }
  ```

  + delete p;
  + p = nullptr;
  + unique_ptr将他们封装成一个操作，只需要p = nullptr，即可，不会保留一个空悬指针
  + ![image-20220925131023725](C++.assets/image-20220925131023725.png)![image-20220925131059994](C++.assets/image-20220925131059994.png)![image-20220925132758108](C++.assets/image-20220925132758108.png)

+ **shared_ptr（原子操作，多线程安全）**：多个指针指向同一个堆空间，通过引用计数(use_counts)，来记录有所有权的指针个数，当引用计数为0时，回收这个堆空间，存在循环引用，空间不释放

+ ```c++
  struct Father
  {
      shared_ptr<Son> son_;
  };
  
  struct Son
  {
      shared_ptr<Father> father_;
  };
  
  int main()
  {
      auto father = make_shared<Father>();
      auto son = make_shared<Son>();
  
      father->son_ = son;
      son->father_ = father;
  
      return 0;
  }
  //1.main 函数退出之前，Father 和 Son 对象的引用计数都是 2。
  //2.son 指针销毁，这时 Son 对象的引用计数是 1。
  //3.father 指针销毁，这时 Father 对象的引用计数是 1。
  //4.由于 Father 对象和 Son 对象的引用计数都是 1，这两个对象都不会被销毁，从而发生内存泄露。
  ```

+ 解决循环引用

+ ```c++
  struct Father
  {
      shared_ptr<Son> son_;
  };
  
  struct Son
  {
      weak_ptr<Father> father_;
  };
  
  int main()
  {
      auto father = make_shared<Father>();
      auto son = make_shared<Son>();
  
      father->son_ = son;
      son->father_ = father;
  
      return 0;
  }
  ```

  

+ ![image-20220914124033815](C++.assets/image-20220914124033815.png)
+ **weak_ptr：**弱引用计数的共享使用权的智能指针，指向同一个空间引用计数不会加1，弱引用的拷贝与析构不会影响其引用计数器
+ 可以通过lock()随时产生一个新的shared_ptr作为强引用，但不lock的时候不影响计数
+ 如果失效（计数器归零）则expired()会返回false，且lock()也会返回nuullptr
+ ![image-20220925141741729](C++.assets/image-20220925141741729.png)

## 完美转发

​	参数转发，例如将参数arg通过factory传递给T的构造

```c++
template <typename T,typename Arg>
shared_ptr<T> factory(Arg arg)			//会有拷贝构造		
{
	return shared_ptr<T>(new T(arg));
}
```

```c++
template <typename T,typename Arg>
shared_ptr<T> factory(Arg& arg)			//不会拷贝构造，但是参数不能为右值
{
	return shared_ptr<T>(new T(arg));
}
```

```c++
template <typename T,typename Arg>
shared_ptr<T> factory(Arg const& arg)	//参数可以为右值，但是当有多个参数时，针对每个形参，既要实现const版本又要实现非												const版本
{
	return shared_ptr<T>(new T(arg));	//本质缺点，参数还是左值，无法触发move语义
}
```

+ 调用wrapper时传递的是左值，内层函数被调用时得到的就是左值
+ 调用wrapper时传递的是右值，内层函数被调用时得到的就是右值

```c++
class CTemp
{
public:
	int m_iData;
public:
	CTemp(int& arg):m_iData(arg){
		std::cout<< "&" << std::endl;
	}
	CTemp(int&& arg):m_iData(arg){
		std::cout << "&&" <<std::endl;
	}
};

template <typename T,typename Arg>
std::shared_ptr<T>factory(Arg&& arg)			
{
	return std::shared_ptr<T>(new T(std::forward<Arg>(arg)));	//完美转发
}
 
int main()
{
	int value = 6;
	auto p1 = factory<CTemp>(6);
	auto p2 = factory<CTemp>(value);
	return 0;
}
```

**std::forward实现原理**

1. 万能引用

   ```c++
   template<typename T>
   void func(T&& arg){
   
   }
   ```

2. 引用折叠

   简化规则

   ```c++
   T& &   -> T&;
   T& &&  -> T&;
   T&& &  -> T&;
   T&& && -> T&&;
   ```

   ```c++
   template<typename T>                            
   void foo(T&& arg){                              //万能引用
       std::cout << "foo(T&& arg)" << std::endl;
   }
   
   int main()
   {
       int value = 6;
       foo(value); //左值，模板T被展开为 int&
       foo(5);     //右值，模板T被展开为 int
       return 0;
   }
   ```

# std::thread

+ 错误：找不到符号pthread_create
+ std::thread是基于pthread
+ ![image-20221017143322933](C++.assets/image-20221017143322933.png)
+ 迅雷下载多线程，下载的时候，可以操作UI界面
+ 主线程等待子线程结束：join()
+ ![image-20221017153701584](C++.assets/image-20221017153701584.png)

**std::thread的析构函数会销毁线程**

+ 作为一个C++类，std::thread同样遵循RAII思想和三五法则：因为管理着资源，他自定义了析构函数，删除了拷贝构造/赋值函数，但是提供了移动构造/赋值函数
+ 因此，当t1所在的函数退出时，就会调用std::thread的析构函数，就会销毁t1线程，t1线程的资源会被销毁，download函数用到的栈也会被销毁，出现错误。

**析构函数不再销毁线程：detach()**

+ 调用成员函数detach()分离该线程——意味着线程的声明周期不再由当前std::thread对象管理，而是在线程退出以后自动销毁自己
+ 但是进程退出时还是会自动销毁
+ ![image-20221017161954876](C++.assets/image-20221017161954876.png)
+ 解决办法1：移动到全局线程池
+ ![image-20221017174319159](C++.assets/image-20221017174319159.png)
+ 解决办法2：main()退出后自动join全部线程
+ ![image-20221017174707293](C++.assets/image-20221017174707293.png)

+ C++20引入了std::jthread类，和std::thread不同在于：他的析构函数里会自动调用join()。

**互斥量**

**std::lock_guard：符合RAII思想的上锁和解锁**

+ 构造函数中会调用mtx.lock()，析构函数中会调用mtx.unlock()。从而退出函数作用域时能够自动解锁
+ ![image-20221017182433175](C++.assets/image-20221017182433175.png)

**std::unique_lock：也符合RAII思想，但自由度更高**

+ std::lock_guard严格在析构时unlock()，但是有时候我们会希望提前unlock()。这时可以用std::unique_lock，它额外存储了一个flag表示是否已经被释放。他会在析构检测这个flag，如果没有释放，则调用unlock()，否则不调用。
+ 可以直接调用unique_lock的unlock()来提前解锁，即使忘记解锁也没关系，退出作用域时候它还会自动检查一遍要不要解锁
+ ![image-20221017183140762](C++.assets/image-20221017183140762.png)
+ 用std::defer_lock作为参数
+ std::unique_lock的构造函数还可以有一个额外参数，那就是std::defer_lock
+ 指定了这个参数的话，std::unique_lock不会在构造函数中调用mtx.lock()，需要之后再手动调用grd.lock()才能上锁
+ 好处依然是即使忘记grd.unlock()也能够自动调用mtx.unlock()。
+ 用std::try_to_lock做参数，和无参数相比，会调用mtx.try_lock()而不是mtx.lock()。之后可以通过grd.owns_lock()判断是否上锁成功
+ ![image-20221017190722626](C++.assets/image-20221017190722626.png)
+ 用std::adopt_lock做参数，如果当前mutex已经上锁了，但是之后仍然希望用RAII思想在析构时自动调用unlock()，可以用std::adopt_lock作为std::unique_lock或std::lock_guard的第二个参数，这时他们会默认mtx已经上锁
+ ![image-20221017191252478](C++.assets/image-20221017191252478.png)

**std::unique_lock和std::mutex具有同样的接口**

+ 其实std::unique_lock具有mutex的所有成员函数：lock()、unlock()、try_lock()、try_lock_for()等。除了他会在析构时自动调用unlock()。
+ 因为std::lock_guard无非是调用其构造参数名为lock()的成员函数，所以std::unique_lock也可以作为std::lock_guard的构造参数

多个对象，每个对象一个mutex即可

**如果上锁失败，不要等待：try_lock()**

+ lock()如果发现mutex已经上锁，会等到它解锁
+ 也可以用无阻塞的try_lock()，上锁失败时返回false，成功返回true
+ ![image-20221017185314934](C++.assets/image-20221017185314934.png)
+ 等待一段时间：try_lock_for()
+ 超过最长等待时间，返回false，如果上锁成功返回true
+ ![image-20221017190040456](C++.assets/image-20221017190040456.png)

**死锁**

+ 保证双方上锁顺序一致
+ 用std::lock同时对多个上锁，接受任意多个mutex作为参数
+ std::lock的RAII版本：std::scoped_lock
+ 即使只有一个线程，在lock()之后又调用lock()，也会造成死锁
+ 可以用recursive_mutex。它会自动判断是不是同一个线程lock()了多次同一个锁，如果是则让计数器+1，之后unlock()会让计数器-1，减到0时才真正解锁，会有性能损失
+ ![image-20221017194255713](C++.assets/image-20221017194255713.png)

**读写锁	**

+ std::shared_mutex
+ ![image-20221017200553130](C++.assets/image-20221017200553130.png)
+ 正如std::unique_lock针对lock()，也可以用std::shared_lock针对lock_shared()。这样就可以在函数体退出时自动调用unlock_shared()了
+ 同样支持defer_lock做参数，owns_lock()判断等

**只需要一次上锁，且符合RAII思想：访问者模式**

![image-20221017202534937](C++.assets/image-20221017202534937.png)

![image-20221017202555811](C++.assets/image-20221017202555811.png)

**条件变量**

+ cv.wait(lck)将会让当前线程陷入等待
+ 在其他线程中调用cv.notify_one()则会唤醒那个陷入等待的线程
+ 还可以额外指定一个参数，变成cv.wait(lck,expr)的形式，其中expr是个lambda表达式，返回值为假时才会阻塞线程，阻塞线程后，只有其返回值为true时才会真正唤醒，否则继续等待。
+ ![image-20221017203705904](C++.assets/image-20221017203705904.png)
+ cv.notify_all()会唤醒全部
+ 这就是为什么wait()需要一个unique_lock作为参数，因为要保证多个线程被唤醒，只有一个能够被启动。
+ ![image-20221017204725743](C++.assets/image-20221017204725743.png)



# STL(标准模板库)

+ 迭代器
+ 容器 ：序列容器（数组、列表、栈、队列）关联容器
+ 算法
+ 空间配置器：一级配置器（malloc）二级配置器（内存池）
+ 配接器：给外面留下了接口，可以改变里面的结构
+ 仿函数：

## **vector**

​	当退出作用域时，自动释放，比较安全。

​	注意当用push_back 向vector 尾部加元素的时候,如果当前的空间不足,vector 会重新申请空间，这次申请的空间是原来的空间大小的1.5（看编译器，gcc是2倍）倍，也就是新的可用空间将增加为原来的1.5倍

​	注意当用pop_back 删除尾部的元素时,vector 的capacity 是不会变化的

+ 插入: O(N)
+ 查看: O(1)
+ 删除: O(N)

**reserve和resize的区别是什么？**

​	reserve只是开辟空间并不创建元素。而resize重新开辟空间并自动初始化元素。

**vector内部实现？**

​	Vector是一个类，它里面有三个指针myfirst,mylast,myend.分别表示首地址，元素容量地址，容器容量地址。通过这三个指针分别表示容器的所有操作。

**优点：**

1. 动态数组，当空间大小不够时，可以自动扩增，每次扩增大小为原来的3倍或者1.5倍。

2. 查询效率很高。

**缺点：**

1. 扩增空间的时候需要重新指向一个连续的内存空间，效率低。

2. 插入删除需要移动元素，效率低。

**vector和list区别**

1. vector底层实现是数组；list是双向链表。
2. vector支持随机访问，list不支持。
3. vector是顺序内存，list不是。
4. vector在中间节点进行插入删除会导致内存拷贝，list不会。
5. vector一次性分配好内存，不够时才进行2倍扩容；list每次插入新节点都会进行内存申请。
6. vector随机访问性能好，插入删除性能差；list随机访问性能差，插入删除性能好。



**初始化**

```c++
std::vector<int> v(8);
//C++11
std::vector<int> v = { 2,0,0,1,0,6,0,6 };
```

```c++
void show(int a)
{
    std::cout << a << std::endl;
}
bool rule(int a,int b)
{
    return a>b;
}
int main()
{
    // vector<int> vec[10];      	//10个数组
    //std::vector<int> vec(4,233);	//4个233的数组
    std::vector<int> vec(10);    //一个数组有10个元素
    srand((unsigned int)time(0));
    // for(int i=0;i<10;i++){
    //     vec.push_back(i);
    // }
    for(int i=0;i<10;i++){
        vec[i] = i;
    }
    random_shuffle(vec.begin(),vec.end());  	//迭代器是算法与容器的桥梁
    sort(vec.begin(),vec.end(),&rule);
    std::cout<< count(vec.begin(),vec.end(),1); //统计1的个数
    for_each(vec.begin(),vec.end(),&show);
}

```

```c++
int main()
{
	std::vector<int> a = {2,0,0,1,0,6,0,6};
	int *p = a.data();							//获取首地址指针
	int n = a.size();
	memset(p,-1,sizeof(int)*n);
	// std::cout << p[0] << std::endl;
	// std::cout << p[1] << std::endl;
	std::cout << a[0] << std::endl;
	std::cout << a[7] << std::endl;
	return 0;
}
```

```c++
int main()
{
	int *p;
	std::vector<int> holder;
	{
		std::vector<int> a = {1,2,3};
		p = a.data();
		holder = std::move(a);			//移动赋值
	}
	std::cout << p[0] << std::endl;
}
```

```c++
//resize:除了可以在构造函数中指定数组的大小，还可以之后再通过resize函数设置大小
int main()
{
	std::vector<int> vec(4,233);
	for(auto num: vec){
		std::cout << num << " ";
	}
	std::cout << std::endl;
	vec.resize(5);
	for(auto num: vec){
		std::cout << num << " ";
	}
	return 0;
}
//clear:清空数组，相当于把长度设置为0，等价于:vec.resize(0)
//reserve:预留一定容量，避免之后重复分配，会移动元素
int main()
{
	std::vector<int> a = {2,0,0,1,0,6,0,6};
	std::cout << a.data() << " " << a.size() << " " << a.capacity() << std::endl;
	a.reserve(15);
	std::cout << a.data() << " " << a.size() << " " << a.capacity() << std::endl;
	a.resize(3);
	std::cout << a.data() << " " << a.size() << " " << a.capacity() << std::endl;
	a.resize(10);
	std::cout << a.data() << " " << a.size() << " " << a.capacity() << std::endl;
	a.resize(15);
	std::cout << a.data() << " " << a.size() << " " << a.capacity() << std::endl;
	return 0;
}
//shrink_to_fit:释放多余容量，只保留刚好为size()大小的容量，会移动元素，迭代器和指针失效
```

```c++
struct Vertex
{
    float x,y,z;
};

std::ostream& operator <<(std::ostream& os,const Vertex& vertex)
{
    os << vertex.x << "," << vertex.y << "," << vertex.z;
    return os;
}

int main()
{
    std::vector<Vertex> vertices;
    vertices.push_back({1,2,3});
    vertices.push_back({4,5,6});
    for(int i=0;i<vertices.size();i++){
        std::cout << vertices[i] << std::endl;
    }
    return 0;
    
}
```

```c++
struct Vertex
{
    float x,y,z;
    Vertex(float x,float y,float z)
        :x(x) , y(y) , z(z)
        {}
    Vertex(const Vertex &vertex)
    {
        std::cout << "copied" << std::endl;
    }
};
int main()
{
    std::vector<Vertex> vertices;
    vertices.push_back(Vertex(1,2,3));
    vertices.push_back(Vertex(4,5,6));
    vertices.push_back(Vertex(4,5,6));
    return 0;
    
}
```

**使用优化**

```c++
struct Vertex
{
    float x,y,z;
    Vertex(float x,float y,float z)
        :x(x) , y(y) , z(z)
        {}
    Vertex(const Vertex &vertex)
    {
        std::cout << "copied" << std::endl;
    }
};
int main()
{
    std::vector<Vertex> vertices;
    // std::vector<Vertex> vertices(3);    //会构造三个Vertex对象   
    vertices.reserve(3);                   //创建3个Vertex大小的内存
    // vertices.push_back(Vertex(1,2,3));  //在main()的当前栈帧中构造一个Vertex对象
    // vertices.push_back(Vertex(4,5,6));  //把这个对象拷贝到vector中
    // vertices.push_back(Vertex(7,8,9));  //vector不断改变大小，里面的对象需要重新拷贝，所以会发生六次拷贝
    vertices.emplace_back(1,2,3);          //在vector内存中使用传进去的构造函数参数列表，创建一个Vertex对象
    vertices.emplace_back(4,5,6); 
    vertices.emplace_back(7,8,9); 
    return 0;                           
    
}
```

emplace 效率更高，没有拷贝构造，直接在vector的内存中创建一个对象



## list

​	是一个双向链表，迭代器具备前移和后移的能力

插入: O(1)

查看: O(N)

删除: O(1)

```c#
void show(int a)
{
    std::cout << a << " ";
}
int main()
{
    std::list<int> lst;
    lst.push_back(0);
    lst.push_back(6);
    lst.push_back(0);
    lst.push_back(6);
    
    std::list<int>::reverse_iterator ite = lst.rbegin();
    while(ite != lst.rend()){
        if(*ite == 0){
            lst.erase(--(ite.base()));                          //将迭代器转为正向的
            break;                                              //正向和反向的迭代器相差一
        }
        ite++;
    }
    for_each(lst.begin(),lst.end(),&show);
    return 0;
}
```

##  map

​	map内部实现了一个红黑树（红黑树是非严格平衡二叉搜索树，而AVL是严格平衡二叉搜索树），红黑树具有自动排序的功能，因此map内部的所有元素都是有序的，红黑树的每一个节点都代表着map的一个元素。因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行的操作。map中的元素是按照二叉搜索树（又名二叉查找树、二叉排序树，特点就是左子树上所有节点的键值都小于根节点的键值，右子树所有节点的键值都大于根节点的键值）存储的，使用中序遍历可将键值按照从小到大遍历出来。

+ 插入: O(logN)

+ 查看: O(logN)

+ 删除: O(logN)

**map优点：**

+ 有序性，这是map结构最大的优点，其元素的有序性在很多应用中都会简化很多的操作

+ map的查找、删除、增加等一系列操作时间复杂度稳定，都为logn

**缺点：**

+ 查找、删除、增加等操作平均时间复杂度较慢，与n相关

**map底层红黑树与AVL**

​	因为avl树是高度平衡，而红黑树通过增加节点颜色从而实现部分平衡，这就导致插入节点两者都可以最多两次实现复衡。而删除节点，红黑树最多三次旋转即可实现复衡，旋转的量级是O(1)，而avl树需要维护从被删除节点到根节点这几个节点的平衡，旋转的量级是O（logn）,所以红黑树效率更高，开销更小，但是因为红黑树是非严格平衡，所以它的查找效率比avl树低。RB-Tree是功能、性能、空间开销的折中结果。

​	实际应用中，若搜索的次数远远大于插入和删除，那么选择AVL，如果搜索，插入删除次数几乎差不多，应该选择RB。

```c++
void show(std::pair<char,int> pr)
{
    std::cout << pr.first << " " << pr.second << std::endl;
}
int main()
{
    std::map<char,int> mp;
    mp['a'] = 6;
    mp['s'] = 0;
    mp['d'] = 2;

    std::map<char,int>::iterator ite = mp.begin();
    while(ite != mp.end())
    {
        std::cout <<  ite->first << " " << ite->second << std::endl;
        ite++;
    }

    mp['a'] = 606;                          //修改
    for_each(mp.begin(),mp.end(),&show);
    
    ite = mp.find('s');
    mp.erase(ite);                          //删除
    for_each(mp.begin(),mp.end(),&show);

    std::pair<char,int> pr('c',2001);       //插入
    mp.insert(pr);
    for_each(mp.begin(),mp.end(),&show);

    std::cout << mp.count('b') << std::endl;//查找是否存在该键值
    return 0;
}
```

## unordered_map

​	unordered_map内部实现了一个哈希表（也叫散列表，通过把关键码值映射到Hash表中一个位置来访问记录，查找的时间复杂度可达到O(1)，其在海量数据处理中有着广泛应用）。因此，其元素的排列顺序是无序的。

+ 查找: O(1)，最坏情况O(N)

+ 删除: O(1)，最坏情况O(N)

**unordered_map优点：**

+ 查找、删除、添加的速度快，时间复杂度为常数级O(c)

**缺点：**

+ 因为unordered_map内部基于哈希表，以（key,value）对的形式存储，因此空间占用率高
+ Unordered_map的查找、删除、添加的时间复杂度不稳定，平均为O(c)，取决于哈希函数。极端情况下可能为O(n)



## set

​	集合（基于红黑树实现，相当于二分查找树）

+ 插入: O(logN)

+ 查看: O(logN)

+ 删除: O(logN)

**set和vector的区别** 

+ 都是能存储一连串数据的容器
+ set会自动给其中的元素从小到大排序，而vector会保持插入时的顺序
+ set会把会把重复的元素去除，只保留一个，即去重
+ vector中的元素在内存上是连续的，可以高效地按索引随机访问，set则不行
+ set中的元素可以高效地按值查找，而vector则低效，查找复杂度是O(log n)



set的排序：string类型会按字典序来排序，所谓字典序就是优先比较两者第一个字符（按ASCII码比较），如果相等则继续比较下一个。



![image-20220711111313834](C++.assets/image-20220711111313834.png)

```c++
int main()
{
	std::set<int> s = {2,0,0,1,0,6,0,6};
	//std::set<int>::iterator s_it = s.begin()+3;	vector提供了+和+=的重载，而set没有。
	std::set<int>::iterator s_it = s.begin();
	++s_it;
	++s_it;
	++s_it;
	std::cout << "s[3] " << *s_it << std::endl;
	return 0;
}
```

**std::next等价于+，支持负数**

```c++
//内部实现
auto next(auto it,int n=1){
	if(it is random_access){
        return it+n;
    }else{
        if(it is bidirectional && n < 0){
            while(n++)
                --it;
        }
    	while(n--)
        	++it;
    	return it;
    }
}

s_it = std::next(s_it,3);
std::cout << "s[3] " << *s_it << std::endl;
```

**std::advance等价于+=，支持负数**

```c++
auto advance(auto &it,int n){
	if(it is random_access){
		it+=n;
    }else{
        if(it is bidirectional && n < 0){
            while(n++)
                --it;
        }
        while(n--)
        ++it;
    }
}

std::advance(s_it,3);
std::cout << "s[3] " << *s_it << std::endl;
```

![image-20220711163932718](C++.assets/image-20220711163932718.png)



**查找元素**

```c++
std::set<int> s = {2,0,0,1,0,6,0,6};
auto it = s.find(2);
std::cout << "2的位置: " << *it << std::endl;
std::cout << "小于2的数: "   << *std::prev(it) << std::endl;
std::cout << "大于2的数: "   << *std::next(it) << std::endl; 
// if(s.find(8) != s.end())	std::cout << "8存在 " << std::endl;
// else	std::cout << "8不存在" << std::endl;
if(s.count(8))	std::cout << "8存在 " << std::endl;
else	std::cout << "8不存在 " << std::endl;
```

**删除元素**

```c++
std::set<int> s = {2,0,0,1,0,6,0,6};
auto it = s.find(2);
s.erase(1);
s.erase(it);
for(auto it = s.begin();it != s.end();it++){
	std::cout << *it << " ";
}
return 0;
```

![image-20220712083953070](C++.assets/image-20220712083953070.png)

**从set中删除指定范围内的元素**

​	erase支持输入两个迭代器作为参数

​	set.erase(beg,end)可以删除集合中从beg到end之间的元素，包含beg,不包含end，[beg,end)。

+ lower_bound(x)：找第一个大于等于x的元素

+ upper_bound(x)：找第一个大于x的元素

```c++
s.erase(s.lower_bound(2),s.upper_bound(4));
//删除set中所有满足2<=x<=4的元素
```

**set和其他容器之间的转换**

​	vector的构造函数也能接受两个前向迭代器作为参数，set的迭代器符合这个要求

```c++
std::set<int> s = {2,0,0,1,0,6,0,6};
std::vector<int> v = {s.lower_bound(1),s.upper_bound(6)}; 
```

​		vector转成set让元素自动排序和去重

```c++
std::vector<int> v = {9,8,5,2,1,1};
std::set<int> s(v.begin(),v.end());
v.assign(s.begin(),s.end());
for(auto value:v){
	std::cout << value << " ";
}
std::cout << std::endl;
```

​	清空set

```c++
s.clear();
s = {};
s.erase(s.begin(),s.end());
```

+ set的不去重版本：multiset

+ equal_range()：一次性求出两个边界，获得等值区间

+ pair<iterator,iterator> equal_range(int const &val) const;

```c++
std::multiset<int> ms = {9,8,5,2,1,1};
auto r = ms.equal_range(1);
ms.erase(1);
for(auto value:ms){
	std::cout << value << " ";
}
return 0;
```

**C++11新增：unordered_set**

​	unordered_set不会排序，里面的元素都是完全随机的顺序，和插入的顺序也不一样，基于哈希散列表实现

+ vector适合按索引查找，通过运算符[]
+ set适合按值相等查找，按值大于/小于查找，分别通过find()、lower_bound()、upper_bound()
+ unordered_set适合按值相等查找，通过find()

