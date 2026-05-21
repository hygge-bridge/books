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





