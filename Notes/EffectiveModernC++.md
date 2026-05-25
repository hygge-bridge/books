# CHAPTER 1 Deducing Types

## Item 1: Understand template type deduction.

*Determine the type of a variable*：

### Case 1: ParamType is a Reference or Pointer, but not a Universal Reference

```c++
int x = 27;
const int cx = x;
const int& rx = x;
const int* px = &x;

template<typename T>
void f(T& param);
template<typename T>
void f_const(const T& param);
template<typename T>
void f_pointer(T* param);

int main()
{
	f(x);
	f(cx);
	f(rx);

	f_const(x);
	f_const(cx);
	f_const(rx);	
	
	f_pointer(&x);
	f_pointer(px);
	return 0;
}
```

### Case 2: ParamType is a Universal Reference

```c++
int x = 27;
const int cx = x;
const int& rx = x;

template<typename T>
void f(T&& param);

int main()
{
	f(x);
	f(cx);
	f(rx);
	f(27);
	return 0;
}
```

### Case 3: ParamType is Neither a Pointer nor a Reference

```c++
const char* const ptr = "Fun with pointers";
template<typename T>
void f(T param);

int main()
{
	f(ptr);
	return 0;
}
```

### Array Arguments

```c++
const char name[] = "J. P. Briggs";
const char* ptrToName = name;

template<typename T>
void f(T param);
template<typename T>
void f_reference(T& param);
template<typename T>
void f_pointer(T* param);

int main()
{
	f(ptrToName);
	f_reference(name);
	f_pointer(name);
	return 0;
}
```

*Get array size at compile time*:

```c++
#include <array>
#include <cstddef> 

template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept
{
	return N;
}

int main()
{
	int keyVals[] = { 1, 3, 7, 9, 11, 22, 35 };
	std::array<int, arraySize(keyVals)> mappedVals;
	return 0;
}
```

### Function Arguments

```c++
void someFunc(int, double);

template<typename T>
void f1(T param);
template<typename T>
void f2(T& param);

int main()
{
	f1(someFunc);
	f2(someFunc);
	return 0;
}
```

## Item 2: Understand auto type deduction

*Determine the type of a variable*：

```c++
#include <initializer_list>
#include <vector>

void someFunc(int, double);

template<typename T>
void f(T param); 
template<typename T>
void fInitList(std::initializer_list<T> initList);

auto createInitList()
{
	return { 1, 2, 3 };
}

int main()
{
	auto x = 27;
	const auto cx = x;
	const auto& rx = x;
	auto&& uref1 = x;
	auto&& uref2 = cx;
	auto&& uref3 = 27;

	const char name[] = "R. N. Briggs";
	auto arr1 = name; 
	auto& arr2 = name;
	auto func1 = someFunc; 
	auto& func2 = someFunc;

	auto x1 = 27;
	auto x2(27);
	auto x3 = { 27 }; 
	auto x4{ 27 };
	auto x5 = { 1, 2, 3.0 };

	f({ 11, 23, 9 });
	fInitList({ 11, 23, 9 });

	std::vector<int> v;
	auto resetV = [&v](const auto& newValue) { v = newValue; }
	resetV({ 1, 2, 3 });

	return 0;
}
```

## Item 3: Understand decltype

*Determine what `decltype` yields for a variable*：

```c++
int main()
{
	const int i = 0;
	bool f(const Widget & w);
	struct Point {
		int x, y;
	};
	Widget w;
	if (f(w)) 
	vector<int> v;
	if (v[0] == 0)
	return 0;
}
```

*Determine the type of a variable*：

```c++
Widget w;
const Widget& cw = w;
auto myWidget1 = cw;
decltype(auto) myWidget2 = cw;
```

*Access an element in a container*:

```c++
#include <vector>

template<typename Container, typename Index>
auto authAndAccess1(Container& c, Index i) -> decltype(c[i])
{
    // ... authenticate the user, check permissions, etc. ...
    return c[i];
}

template<typename Container, typename Index>
auto authAndAccess2(Container& c, Index i)
{
    // ... authenticate the user, check permissions, etc. ...
    return c[i];
}

template<typename Container, typename Index>
decltype(auto) authAndAccess3(Container& c, Index i)
{
    // ... authenticate the user, check permissions, etc. ...
    return c[i];
}

// c++14 version
template<typename Container, typename Index>
decltype(auto) authAndAccess4(Container&& c, Index i)
{
    // ... authenticate the user, check permissions, etc. ...
    return std::forward<Container>(c)[i];
}

// c++11 version
template<typename Container, typename Index>
auto authAndAccess5(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i])
{
    // ... authenticate the user, check permissions, etc. ...
    return c[i];
}

int main()
{
    std::vector<int> v{ 0, 1 };
    authAndAccess1(v, 1) = 3;
    authAndAccess2(v, 1) = 3;
    authAndAccess3(v, 1) = 3;
    authAndAccess4(std::vector<int>(1, 2), 1);
    authAndAccess5(std::vector<int>(1, 2), 1);
	return 0;
}
```

*Determine what type decltype evaluates to*:

```c++
decltype(auto) f1()
{
	int x = 0;
	return x;
}
decltype(auto) f2()
{
	int x = 0;
	return (x);
}

int main()
{
	int x = 0;
    decltype(x) y = x;
	decltype((x)) z = x;
	return 0;
}
```

## Item 4: Know how to view deduced types

*Multiple ways to view a variable's deduced type*:

1. IDE Editors: hover your cursor over the entity (may be neither helpful nor accurate)

2. Compiler Diagnostics: cause the compiler to report an error, thereby examining the type

   ```c++
   template<typename T>
   class TD;
   
   int main()
   {
   	const int theAnswer = 42;
   	auto x = theAnswer;
   	auto y = &theAnswer;
   	TD<decltype(x)> a;
       TD<decltype(y)> b;
   	return 0;
   }
   ```

3. Runtime Output： `boost::type_id_with_cvr<T>()`and`std::type_info::name()` may be neither helpful nor accurate

   ```c++
   #include <iostream>
   #include <vector>
   
   template<typename T>
   void f(const T& param)
   {
   	std::cout << typeid(T).name() << '\n';
       std::cout << typeid(param).name() << '\n';
   }
   
   int main()
   {
   	const int theAnswer = 42;
   	auto x = theAnswer;
   	auto y = &theAnswer;
   	std::cout << typeid(x).name() << '\n';
   	std::cout << typeid(y).name() << '\n';
   
   	const std::vector<int> v;
   	f(&v);
   	return 0;
   }
   ```

4. The understanding of C++’s type deduction rules remains essential!

# CHAPTER 2 auto

## Item 5: Prefer auto to explicit type declarations

##### *What are the advantages of using `auto` instead of the original way writing it?*

```c++
// Declare a normal variable.
int x1;
auto x2;
auto x3 = 0;
```

1. `x1` isn't initialized , so its value is indeterminate -- or it is initialized to zero, depending on the context.
2. `auto` variables have their type deduced from the initializer, so they must be initialized. That means they(`x2 x3`) eliminate uninitialized problems.

```c++
// Iterate over the contents between the iterators.
template<typename It>
void iter1(It b, It e)
{
    while (b != e) {
        typename std::iterator_traits<It>::value_type currValue = *b;
    }
}
template<typename It>
void iter2(It b, It e)
{
    while (b != e) {
    	auto currValue = *b;
    }
}
```

1. Instead of writing `typename std::iterator_traits<It>::value_type currValue = *b;`, using `auto` would be much less typing.

```c++
// Compare the values of the elements pointed to by the pointers.
std::function<bool(const std::unique_ptr<int>&, const std::unique_ptr<int>&)>
compFunc1 = [](const std::unique_ptr<int>& p1, const std::unique_ptr<int>& p2)
	{
		return *p1 < *p2;
	};
auto compFunc1 = [](const std::unique_ptr<int>& p1,const std::unique_ptr<int>& p2)
    { 
        return *p1 < *p2; 
    };
auto compFunc2 = [](const auto& p1,const auto& p2)
    { 
        return *p1 < *p2; 
    };
```

 

## Item 8: Prefer nullptr to 0 and NULL

函数重载

清晰

模版

## Item 9: Prefer alias declarations to typedefs

函数指针 			

模版（normal template -> type traits）

## Item 10: Prefer scoped enums to unscoped enums

作用域

隐式转换（特定场景下，这东西确实有用）

前置声明（C++11下其实unscoped enum也支持了）

## Item 11: Prefer deleted functions to private undefined ones

报错时机

对于类的报错信息

类中函数模版特化

## Item 12: Declare overriding functions override

让编译器报错

修改基类虚函数

补充：成员函数的引用限定符

## Item 13: Prefer const_iterators to iterators.

只要不改数据，尽量用const，值传递就无所谓了

为了最大泛化，使用非成员函数begin

补充：在c++11中实现cbegin

## Item 14: Declare functions noexcept if they won’t emit exceptions

c++11和c++98在声明为无异常但是扔出异常的区别

noexcept 带来的性能提升

标准库的swap的noexcept的特殊点

内存释放和析构函数的noexcept

接口设置何时添加noexcept

## Item 15: Use constexpr whenever possible

constexpr 和const的区别

## Item 16: Make const member functions thread safe

const成员方法在多线程下的问题

什么场景下使用哪一种方案

## Item 17: Understand special member function generation

default construct, copy construct/assignment, move construct/assignment, deconstruct的编译器自动生成时机

成员函数模型的影响

关于编译器自动生成函数的最佳实践写法

# CHAPTER 4 Smart Pointers

裸指针的问题

## Item 18: Use std::unique_ptr for exclusive-ownership resource management

unique_ptr特点：

1. 效率
2. 排他（只move）
3. 自定义deleter（效率）
4. 转shared_ptr很简单
5. 指向对象和数组的区别（但是除非是有个c库返回了一个c指针，否则用stl的数组）

主要的两个用途：

1. 工厂函数
2. pimpl

## Item 19: Use std::shared_ptr for shared-ownership resource management

动机：垃圾回收只能回收内存，c++需要通用性和可预测性

引用计数导致的shared_ptr开销：

1. 两个指针（一个指向数据+一个指向控制块）
2. 控制块需要额外的动态分配内存，make_shared可以解决，但是make_shared无法自定义deletor（也就是有些场景下不能用）
3. 引用计数的增减是原子的，所以性能比直接增减慢一点

创建控制块的时机：

1. make_shared
2. 由unique_ptr创建的shared_ptr
3. 裸指针创建的shared_ptr

不要直接对裸指针构建shared_ptr，直接new一个，另外的用原来的shared_ptr。但是对于this怎么办？ ---> 继承enable_shared_from_this，私有化构造。

## Item 20: Use std::weak_ptr for std::shared_ptr-like pointers that can dangle

在可能有悬挂指针的场景下使用weak_ptr，常见的场景有：

1. 缓存
2. 观察者列表
3. 避免shared_ptr循环（一般层级结构不会有循环引用的问题，因为parent拥有unique_ptr的child，child可以使用raw pointer来使用parent，因为child的生命周期一般都比parent短）

性能和shared_ptr差不多。它只是不参与共享拥有权所以不会影响被指向对象的引用计数。但是他会影响控制块的弱引用计数。

weak_ptr升级为shared_ptr是原子操作

```c++
std::shared_ptr<int> sp;
std::weak_ptr<int> wp{sp};
// 升级都是原子操作
auto sp2 = wp.lock();
std::shared_ptr<int> sp3{wp};
```

## Item 21: Prefer std::make_unique and std::make_shared to direct use of new

make函数的优点：

1. dry原则

   ```c++
   auto spw1(std::make_shared<Widget>()); // 只写了一次Widget
   std::shared_ptr<Widget> spw2(new Widget); // 写了两次Widget
   ```

2. exception-safe（函数参数求值顺序无法确定导致的问题）

   ```c++
   void processWidget(std::shared_ptr<Widget> spw, int priority);=
   // 有可能在《new》后《构造shared_ptr》前，执行computePriority，但是抛出异常，new的内存就无法释放了
   processWidget(std::shared_ptr<Widget>(new Widget), computePriority()); 
   // new和构造被包装成一次操作
   processWidget(std::make_shared<Widget>(), computePriority());
   ```

3. 效率更高，object code更小更高效。

   1. make_shared的控制块和对象内存是连续内存，所以无需在控制块存储指向对象内存的指针
   2. make_shared只会分配一次内存



缺点：

1. make函数不支持自定义删除器

2. make函数内部使用完美转发，而`{}`无法被完美转发（因为{}没有类型），所以当想要使用初始化列表时，只能先够着初始化列表，再传递给make函数

   ```c++
   auto initList = { 10, 20 };
   auto spv = std::make_shared<std::vector<int>>(initList);
   // 错误
   auto spv = std::make_shared<std::vector<int>>({ 10, 20 });
   ```

3. 自定义operator new/delete时，因为自定义new/delete一般只会考虑对象的size，但是shared_ptr还有控制块的size，所以使用make函数会有问题

4. 当weak_ptr比shared_ptr声明周期长，且对象内存较大时，且系统堆内存敏感。会导致内存长时间没有释放，从而导致内存一直减不下去。因为控制块还有弱引用计数，当强引用计数为0时，释放对象内存，但是make函数将对象和控制块内存分配在一起了，所以必须强引用和弱引用都为0，才会释放。



确保异常安全且性能提升小技巧：使用移动语义，然后在函数调用前就将智能指针构造好

```c++
std::shared_ptr<Widget> spw(new Widget, cusDel);
// 异常安全，因为spw在computePriority可能抛出异常前已经构造了
processWidget(std::move(spw), computePriority());
```

## Item 22: When using the Pimpl Idiom, define special member functions in the implementation file

使用unique_ptr实现pimpl：

- 优点：自动资源管理
- 缺点：即使是使用编译器自动生成的特殊函数，也必须在头文件声明，在源文件中实现。理由如下，
  - unique_ptr的删除器是指针内部的结构，所以当进行析构时需要知道完整的类型，其默认删除器会使用static_assert判断指针是否是完整类型，如果不是就会编译报错。
  - 编译器默认生成的版本是自动内联的，然而头文件中没有指针的完整类型。

注意：因为unique_ptr不支持拷贝，所以如果指针指向的类型是支持拷贝的，我们的类也应该支持拷贝，而且一般为深拷贝，所以我们一般需要手动实现copy函数，而不能使用默认创建的。

使用shared_ptr实现pimpl：

- 优点：无需在头文件声明源文件中实现（shared_ptr的删除器不是指针的内部结构）
- 缺点：性能开销大

总结，仍然是根据所有权判断选择智能指针：

- 在独占所有权下，使用unique_ptr（一般pimpl都是独占的）
- 在共享所有权下，使用shared_ptr



# CHAPTER 5 Rvalue References, Move Semantics, and Perfect Forwarding

## Item 23: Understand std::move and std::forward

move和forward本质都是类型转换

move：无条件转为右值

```c++
template<typename T>
decltype(auto) move(T&& param)
{
    using ReturnType = remove_reference_t<T>&&;
	return static_cast<ReturnType>(param);
}
```

forward：当传入的参数是右值才转为右值（因为参数默认是左值）

注意：对于移动操作，参数不应该设置为const，因为移动就是要破坏原有的结构，且设置为const会导致调用到拷贝函数而不是移动函数，因为右值版本的函数无法接受const参数。

forward理论上可以替代move，但是实际上不行的原因：

1. forward的语义不明确，且写的更多更麻烦`std::forward<std::string>(rhs.s) == std::move(rhs.s)`
2. 如果写错类型，编译没有问题，只是导致最后调用copy而不是move，难以调试`std::forward<std::string&>(rhs.s)`

## Item 24: Distinguish universal references from rvalue references

万能引用用户传入左值就是左值，右值就是右值。右值引用无论传入什么，都是右值。

- 万能引用：有类型推导，格式为`type&&`或者`auto&&`
- 右值引用：没有类型推导的`type&&`，或者格式不是`type&&`，比如`void f(const T&& param);`是右值应用

举例：push_back是右值引用，因为T的类型在构造vector时已经确定了，而emplace_back是万能引用

```c++
template<class T, class Allocator = allocator<T>> // from C++
class vector { // Standards
public:
    void push_back(T&& x);
    
    template <class... Args>
	void emplace_back(Args&&... args);
}
```

## Item 25: Use std::move on rvalue references, std::forward on universal references
