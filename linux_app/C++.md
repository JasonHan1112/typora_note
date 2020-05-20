

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
  
- 数组指针，指针数组，数组引用

  ```c
  int *ptrs[10];//是一个数组，数组的元素是10个指向整型数据的指针
  int (*parray)[10];//是一个指针，指向了一个10个整型元素的指针
  int (&arr_ref)[10];//arr_ref引用一个含有10个整型的数组
  ```

- 标准库函数begin和end
  ```c
  int ia[] = {0, 1, 2, 3, 4};
  int *ia_beg = begin(ia); //指向ia首元素的指针
  int *ia_lst = end(ia); //指向arr尾元素的下一位置的指针
  ```
  
- 数组初始化vector
  ```c
  int int_arr[] = {1, 2, 3, 4, 5, 6};
  vector<int> ivec(begin(int_arr), end(int_arr));
  ```
  
- 范围for语句
  ```c
  vector<int> v = {0, 1, 2, 3, 4, 5, 6};
  for(auto &r : v) {
  ...
  }
  //以下为等价的传统for语句
  for(auto beg = v.begin(), end = v.end(); beg != end; ++beg) {
    ...
  }
  //不能通过范围for语句添加vector元素，因为其中的end元素已经预存，添加元素后会引起混乱
  ```
  
- try语句和异常处理
  ```c
  try {
    /*throw*/
    /*后边的catch会抓到 thow的值*/
  } catch (exception) {
    /*handler*/
  } catch (exception) {
    /*handler*/
  }//
  ```
  
- C++建议函数的形参用引用代替指针

- 函数重载
  函数名称相同但是形参列表不同称之为函数重载。
  
- 友元
  
  - 类可以允许其他类或者函数访问其非共有成员，方法是声明该函数成为它的友元。
  
  ```c
  class Sales_data {
      friend Sales_data add(const Sales_data&, const Sales_data&);
      friend std:istream &read(std::istream&, Sales_data&);
      friend std::ostream &print(std::ostream&, const Sales_data&);
      public:
          ...
      private:
          ....
  };
  Sales_data add(const Sales_data&, const Sales_data&);
  std:istream &read(std::istream&, Sales_data&);
  std::ostream &print(std::ostream&, const Sales_data&);
  ```
  
  - 友元的声明仅仅制定了访问权限，并非是一个通常意义上的函数声明，如果希望类的用户能够访问一个友元，需要在友元声明之外再对函数做一次声明。
  
  - inline可以在声明的时候指定，也可以在定义的时候指定
  
- 委托构造函数
  委托构造函数是使用它所属类的其他构造函数执行它自己初始化过程。
  ```c
  class sales_data {
      public:
          //非委托
          sales_data(std::string s, unsigned cnt, double price):
              bookNo(s), units_sold(cnt), revenue(cnt*price) {}
          //委托第一个构造函数完成
          sales_data(): sales_data("", 0, 0) {}
          sales_data(std::string s): sales_data(s, 0, 0) {}
          sales_data(std::istream &is): sales_data()
                                           {read(is, *this);}
          
  }
  ```
- 聚合类
  - 所有成员都是public
  - 没有定义任何构造函数
  - 没有类内初始值
  - 没有基类，没有virtual函数
  - 类似C语言的结构体
  ```c
  struct data {
    int ival;
    string s;
  }
  ```
- 静态成员变量
  ```c
  class account {
    public:
        static double rate() { return intrerst_rate; }
        static void rate(double);
  }
  ```
  - 静态成员变量不和任何对象绑定在一起，不包含this指针。不能在static函数体内使用this指针
  - 使用静态成员变量，通过作用与运算符访问静态成员(account::rate();)
  - 成员函数不用通过作用域运算就能直接使用静态成员

- C++标准库
- 容器类型
  - vector: 可变大小数组，支持快速访问，在尾部插入或删除元素很快，其他地方很慢（连续空间）
  - deque: 双段队列，在头尾插入或删除很快
  - list: 双向链表，只支持顺序访问，在任何位置插入删除操作都很快
  - forwad_list: 单项链表，只支持顺序访问，在任何位置插入删除操作都很快
  - array: 固定大小数组，支持快速随机访问，不能添加或删除元素
  - string: 与vector类似，专门用于保存字符（连续空间）
  
  ```c
  list<sales_data> a;
  list<double> c;
  vector<int> d;
  ```
- 迭代器
  一般通过xxx.begin()获得
- 容器中添加元素
  - 每一个顺序容器都支持xxx.push_back(xxx);[在尾端插入数据]
  - 在指定位置插入元素
    ```c
    xxx.insert(iter, "hello"); //将“hello”添加到iter之前
    ```
  - emplace_back
  在容器管理的内存空间中直接构建元素，没有拷贝对象到容器的过程。（调用对象的构造函数）
  ```c
  c.emplace_back("hello", a, b);//参数要和构造函数的参数一致
  ```
  - 删除元素
    - pop_front, pop_back: 删除首元素和为尾元素，不是所有容器都支持
    - xxx.clear(); //删除所有元素
    - xxx.erase(elem1, elem2); //elem1指向要删除的第一个元素， elem2指向我们要删除的最后一个元素的后一个元素。
- 智能指针
  - share_ptr
    - 智能指针类似vector，也是模板，因此当我们创建一个只能指针时，必须提供一个类型。
    ```c
    shared_ptr<string> p1; //指向string的shared_ptr
    ```
    - 默认初始化的智能指针保存着一个空指针
    ```c
    //如果p1不为空，检查它是否只想一个空的string
    if(p1 && p1->empty())
        *p1 = "hi";
    ```
  - unique_ptr
    - 独占所指向的对象
  - make_shared函数
    - 最安全的分配和使用动态内存的方法。动态内存中分配一个对象并初始化，返回指向此对象的shared_ptr
    ```c
    shared_ptr<int> p3 = make_shared<int>(42); //指向一个值为42的int的shared_ptr
    shared_ptr<string> p4 = make_shared<string>(10, '9');//"9999999999"
    ```
    - 一般用auto变量来保存make_shared的结果
    - 每个shared_ptr都有一个关联的计数器，通常称为引用计数，无论何时拷贝一个shared_ptr都会递增。当引用计数变为0,就会自动释放管理对象。
  - shared_ptr和new结合使用
    ```c
    shared_ptr<double> p1;
    shared_ptr<int> p2(new int(42));
    ```
- 面向对象
  - 基类
  ```c
  class quote {
      public:
          std::string isbn() const;
          virtual double net_price(std::size_t n) const;
  };
  ```
  - 派生类
    class bulk_quote: public quote {
        public:
            double net_price(std::size_t) const override;
    };
  - 虚函数
    基类声明成虚函数，子类来实现该函数
    ```c
    class quote {
        public:
            xxx
            virtual souble net_price(std::size_t n) const;
    }
    ```














