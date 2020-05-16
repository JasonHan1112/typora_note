

# C++ Note
- 命名空间
  - std::cout, std::cin等等，其中std表示命名空间，命名空间可以帮助我们避免不经意间的名字冲突。
  - ::表示作用域运算符
  - using声明命名空间 using namespace::name
    ```c
    using namespace std;
    #include <iostream>
    int main()
    {
        int i;
        cin << i;
        return 0;
    }
    ```
  - 头文件不应该有using namespace，因为有可能会多次包含并造成名字冲突。
- std::endl 和 \n不同，两者都有换行的意思但是std::endl还有刷新输入流的作用。
- std::cin 接收到文件结束符之后会变为假（while(std::cin>>a)）windows中文件结束符**Ctrl+Z**，Linux中包括Mac OS中文件结束符**Ctrl+D**
- 显示的访问全局变量：通过::运算符例如： ::global_var = 3;
- 引用
  - 为对象其了一个另外的名字，通过将声明符写成&d的形式来定义引用类型，其中d是声明的变量名，所以引用只有声明，没有定义。
  ```c
  int ival = 1024;
  int &ref_val = ival;
  int &ref_val1;//报错，引用必须初始化。
  ref_val = 2;//相当于把2赋给了ival。
  int &ref_val2 = ref_val;//ref_val2绑定到了ival上
  int i = ref_val;//相当于i被初始化为ival的值
  ```
  - 一旦初始化完成，引用将和它的初始值一直绑定在一起，因此引用必须初始化。
  - 引用变量不是对象，只是为一个已经存在的变量的一个**别名**。
  - 引用变量只需要在定义的时候有**&**使用时不需要该符号。
  - 引用主要解决函数参数传递中大块数据不拷贝的问题，与指针功能类似。
  - 声明引用后，引用本身不占据存储单元，系统不给引用分配存储空间，因此&ref_val与&ival相等。
  - 引用只能与对象进行绑定，不能与常量进行绑定。
  ```c
  int &refval4 = 10;//错误，引用类型必须是一个对象
  double dval = 3.14;
  int &ref_val5 = dval;//错误，类型要相对应
  ```
  - cont的引用
  ```c
  const int ci = 1024;
  const int &r1 = ci;//正确
  r1 = 42;//错误
  int &r2 = ci;//错误，非常量引用不能绑定常量的引用
  int i = 42;
  const int &r2 = i;//正确，允许将const int &绑定到普通的int上
  const int &r3 = 42;//正确，r3是一个常量引用
  ```
  - 指向指针的引用
  ```c
  int i = 42;
  int *p;
  int *&r = p;//r是一个与整型指针绑定的引用
  r = &i;
  *r = 0;
  ```
- 某符号的多重含义
& 和*在不同阶段表达的含义不同
```c
int i = 42;//定义一个整型变量
int &r = i;//定义一个引用，并初始化绑定为i变量
int *p = &i;//定一个指针，并初始化指向i变量
int *d;//定义一个指针，没有初始化
d = &i;//赋值指向i变量
*d = 3;//将i变量的值赋值为3
int &r2 = *p;//引用与i绑定
```
- C++中nullptr来表示空指针。
- 定义多个变量，int *p, p2;（表示定义一个p的int指针变量，和一个p2的整型变量）
- 常量表达式(constexpr变量)，在编译时就计算完结果
```c
constexpr int mf = 20;//20为常量表达式
constexpr int limit = mf+1;//mf+1是常量表达式
constexpr int sz = size();//只有size()是一个常量表达式函数是才是正确的
```

- const修饰的变量不会被分配存储空间，而是将其保存在符号表中，使其成为编译期间的一个常量

- auto类型

  - C++11新标准，引入auto类型，用它能让编译器替我们分析表达式所属的类型。auto让编译器通过初始值来推断变量的类型，因此auto必须有初值。

    ```c
    auto item = val1 + val2;
    auto i = 0, *p = &i;
    auto sz = 0, pi = 3.14;
    ```
    
  - auto会忽略const，如果需要需要明确指出 const auto f = ci;
  
- 标准库string
  ```c
  #include<string>
  using std::string;
  int main()
  {
      string s1(10, 'h');
      string s2(s1);
      string s3("ssss");
      string s4 = "value";
      return 0;
  }
  ```
  拷贝初始化：如果使用’=‘初始化一个变量，执行的是拷贝初始化。
  直接初始化：如果不使用’=‘初始化一个变量，执行的是拷贝初始化。
- 标准库类型vector
  vector容纳这其他对象，所以它也被称作容器(container)
  ```c
  #include <vector>
  using std::vector;
  ```
  - C++中有类模板也有函数模板，其中vector是一个**类模板**。
  - 模板本身不是类或者函数，模板被看作是为编译器生成类或函数编写的一份说明，编译器根据模板创建类或者函数的过程成为实例化，当使用模板时，需要指出编译器应把类或函数实例化成何种类型。
  ```c
  vector<int> ivec;//ivec保存int类型的对象
  vector<Sales_item> Sales_vec;//保存Sales_item类型的对象
  //初始化vector对象方法
  vector<T> v1; //v1是空的vector，潜在元素是T类型，执行默认初始化
  vector<T> v2(v1); //v2中包含v1中所有元素的副本
  vector<T> v2=v1; //等价于v2（v1）
  vector<T> v3(n, val); //v3包含了n个重复的元素，每个元素值都是val
  vector<T> v4(n); //包含了n个重复的执行了值初始化的对象
  vector<T> v5{a, b, c...}; //包含了初始值a,b,c...每个元素被赋予相应的初始值
  
  vector<string> v1 = {"a", "an", "the"};//拷贝初始化
  vector<string> v2{"a", "an", "the"};//列表初始化
  ```
  - 向vector中添加元素(push_back)，vector不能通过下标添加元素，但是可以通过下标来寻找元素
    ```c
    string word;
    vector<string> text;
    while(std::cin >> word) {
      text.push_back(word);
    }
    ```
  - 其他vector操作
  ```c
  v.empty() //如果v不包含任何元素，返回true，否则返回false
  v.size() //返回v中元素的个数
  ...
  ```
- 迭代器
  string vector支持迭代器，迭代器+n（-n）后也为迭代器
  
  ```c
  /*定义迭代器，vector<int>::iterator it1(可以读写vector<int>元素的迭代器)*/
  /*定义迭代器，string::iterator it2(可以读写string元素的迭代器)*/
  /*为了方便一般会用auto来定义迭代器*/
  std::string s("xxxxxxxx");
  //s.begin()表示地一个元素，s.end()表示尾元素的下一个位置
  for(auto it = s.begin(); it != s.end() && !isspace(*it); ++it) {
    *it = toupper(*it);//将当前字符改成大写
  }
  ```
  
























