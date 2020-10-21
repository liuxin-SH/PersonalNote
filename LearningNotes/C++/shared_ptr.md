# C++智能指针 shared_ptr
shared_ptr 是一个标准的共享所有权的智能指针, 允许多个指针指向同一个对象.   
要确保用 new 动态分配的内存空间在程序的各条执行路径都能被释放是一件麻烦的事情。  
C++ 11 模板库的 <memory> 头文件中定义的智能指针，即 shared _ptr 模板，就是用来部分解决这个问题的。

shared_ptr 是为了解决 auto_ptr 在对象所有权上的局限性(auto_ptr 是独占的), 在使用引用计数的机制上提供了可以共享所有权的智能指针, 当然这需要额外的开销:  
　　(1) shared_ptr 对象除了包括一个所拥有对象的指针外, 还必须包括一个引用计数代理对象的指针.  
　　(2) 时间上的开销主要在初始化和拷贝操作上, *和->操作符重载的开销跟auto_ptr是一样.  

使用方法:
- 可以使用模板函数 make_shared 创建对象,如: 
``` 
std::shared_ptr<int> sp1 = std::make_shared<int>(10);  
std::shared_ptr<std::string> sp2 = std::make_shared<std::string>("Hello c++");
```

- 也可以直接使用如下形式：  
```
shared_ptr<T> ptr(new T);  // T 可以是 int、char、类等各种类型
```

 

## 成员函数
1. use_count 返回引用计数的个数
2. unique 返回是否是独占所有权( use_count 为 1)
3. swap 交换两个 shared_ptr 对象(即交换所拥有的对象)
4. reset 放弃内部对象的所有权或拥有对象的变更, 会引起原有对象的引用计数的减少
5. get 返回内部对象(指针), 由于已经重载了()方法, 因此和直接使用对象是一样的.如 shared_ptr<int> sp(new int(1)); sp 与 sp.get()是等价的.  
6. 智能指针转换不能通过类型强壮进行转换，必须通过库提供转换函数进行转换。  
    C++11的方法是：std::dynamic_pointer_cast.   
    父类必须是多太类型既 virtual ~B();
```C++
std::dynamic_pointer_cast<const SubClass>(pB)
```

以下代码演示各个函数的用法与特点:
```C++
    std::shared_ptr<int> sp0(new int(2));
    std::shared_ptr<int> sp1(new int(11));
    std::shared_ptr<int> sp2 = sp1;
    printf("%d\n", *sp0);               // 2
    printf("%d\n", *sp1);               // 11
    printf("%d\n", *sp2);               // 11
    sp1.swap(sp0);
    printf("%d\n", *sp0);               // 11
    printf("%d\n", *sp1);               // 2
    printf("%d\n", *sp2);               // 11

    std::shared_ptr<int> sp3(new int(22));
    std::shared_ptr<int> sp4 = sp3;
    printf("%d\n", *sp3);               // 22
    printf("%d\n", *sp4);               // 22
    sp3.reset();                        
    printf("%d\n", sp3.use_count());    // 0
    printf("%d\n", sp4.use_count());    // 1
    printf("%d\n", sp3);                // 0
    printf("%d\n", sp4);                // 指向所拥有对象的地址
    
    std::shared_ptr<int> sp5(new int(22));
    std::shared_ptr<int> sp6 = sp5;
    std::shared_ptr<int> sp7 = sp5;
    printf("%d\n", *sp5);               // 22
    printf("%d\n", *sp6);               // 22
    printf("%d\n", *sp7);               // 22
    printf("%d\n", sp5.use_count());    // 3
    printf("%d\n", sp6.use_count());    // 3
    printf("%d\n", sp7.use_count());    // 3
    sp5.reset(new int(33));                        
    printf("%d\n", sp5.use_count());    // 1
    printf("%d\n", sp6.use_count());    // 2
    printf("%d\n", sp7.use_count());    // 2
    printf("%d\n", *sp5);               // 33
    printf("%d\n", *sp6);               // 22
    printf("%d\n", *sp7);               // 22
```
## shared_ptr 的赋值构造函数和拷贝构造函数:
　　auto r = std::make_shared<int>(); // r 的指向的对象只有一个引用, 其 use_count == 1
　　auto q = r; (或auto q(r);) // 给 r 赋值, 令其指向另一个地址, q 原来指向的对象的引用计数减1(如果为0, 释放内存), r指向的对象的引用计数加1, 此时 q 与 r 指向同一个对象, 并且其引用计数相同, 都为原来的值加1.
以下面的代码测试:
```C++
    std::shared_ptr<int> sp1 = std::make_shared<int>(10);
    std::shared_ptr<int> sp2 = std::make_shared<int>(11);
    auto sp3 = sp2; 或 auto sp3(sp2);
    printf("sp1.use_count = %d\n", sp1.use_count());  // 1
    printf("sp2.use_count = %d\n", sp2.use_count());  // 2
    printf("sp3.use_count = %d\n", sp3.use_count());  // 2
    sp3 = sp1;
    printf("sp1.use_count = %d\n", sp1.use_count());  // 2
    printf("sp2.use_count = %d\n", sp2.use_count());  // 1
    printf("sp3.use_count = %d\n", sp3.use_count());  // 2
```

##  Demo实例
```C++
typedef struct _StructA StructA;
typedef struct _StructB StructB;

struct _StructA
{
    int i;
    std::shared_ptr<StructB> pStructB;
    _StructA(): i(int())
    {
        printf("_StructA alloc \n");
    };
    
    ~_StructA()
    {
        printf("_StructA dealloc \n");
    };
} ;

struct _StructB
{
    int i;
    std::shared_ptr<StructA> pStructA;
    _StructB(): i(int())
    {
        printf("_StructB alloc \n");
    };
    
    ~_StructB()
    {
        printf("_StructB dealloc \n");
    };
};
```

### make_shared 和 shared_ptr 的用处
```C++
// make_shared 和 shared_ptr 设计初衷是引入引用计数这个概念来减少程序
// 员的工作量. 他的设计想法跟 Objective-c 的引用计数有点类似。请看以下代码: 

int main(int argc, const char * argv[])
{
    printf("----- start ----- \n");       
    std::shared_ptr<StructA> pA = std::make_shared<StructA>();
    std::shared_ptr<StructB> pB (new StructB());
    printf("----- end ----- \n");
    return 0;
}    


// 打印结果  
**----- start ----- **  
**_StructA alloc **  
**_StructB alloc **  
**----- end ----- **  
**_StructB dealloc **  
**_StructA dealloc **  
```

通过实验可以看出, 超出作用域之后就会对 shared_ptr 所作用的对象进行引用计数减少1,  
如果发现 shared_ptr 所作用的对象引用计数为0则说明，这个对象需要释放内存.  

### 环形引用
```C++
    printf("----- start ----- \n");
    std::shared_ptr<StructA> pA = std::make_shared<StructA>();
    std::shared_ptr<StructB> pB = std::make_shared<StructB>();
    pA->pStructB = pB;
    pB->pStructA = pA;
    printf("----- end ----- \n");

// 打印结果
**----- start ----- **
**_StructA alloc **
**_StructB alloc **
**----- end ----- **
```
环形应用: 就是对象 A 持有对象 B 的强引用, 对象 B 持有对象 A 的强应用,最终导致 A 和 B 都无法释放。  

 解决方法, 其中一方使用弱引用
 ```C++
 struct _StructB
{
    int i;
    // 改为弱引用, 同理针对  StructA 也可以这样做
    std::weak_ptr<StructA> pStructA;
    // std::shared_ptr<StructA> pStructA;
    _StructB(): i(int())
    {
        printf("_StructB alloc \n");
    };
    
    ~_StructB()
    {
        printf("_StructB dealloc \n");
    };
};
```

### make_shared 和 shared_ptr 区别
```C++
   /**
     * 1. 执行申请 数据体(StructA) 的内存申请
     * 2. 执行控制块的内存申请
     */
    std::shared_ptr<StructA> pA1(new StructA());
    
    /**
     * 数据体 和 控制块的 内存一块申请
     */
    std::shared_ptr<StructA> pA2 = std::make_shared<StructA>();
```

### weak_ptr
weak_ptr 主要是用来判断 shared_ptr 所指向的数据内存是否存在,  
因为make_shared 只作一次内存分配, shared_ptr 可以把这种内存分配分为两个步骤,  
weak_ptr 可以通过 lock 来判断 shared_ptr 所指向的数据内存是否被释放.

```C++
    std::shared_ptr<StructA> pSA(new StructA());
    std::weak_ptr<StructA> wPSA = pSA;
    pSA.reset(new StructA());
    auto p = wPSA.lock();
    std::cout<< p << std::endl;
    std::cout<< wPSA.use_count() << std::endl;

// 打印结果
**_StructA alloc **
**_StructA alloc **
**_StructA dealloc **
**0x0**
**0**
**_StructA dealloc **
```

#### weak_ptr 使用注意
```C++
    // 主线程
    std::shared_ptr<StructA> pSA(new StructA());
    std::weak_ptr<StructA> wPSA(pSA);
    
    // 子线程 1
    pSA.reset(new StructA());
    
    // 子线程 2
    // 错误做法, 现在编译器好像也不允许这么做
//    StructA *p = wPSA.get();
//    if ( p )
//    {
//        p->i = 5;
//    }
    
    // 正确做法
    if ( auto p = wPSA.lock() )
    {
        p->i = 5;
    }
```