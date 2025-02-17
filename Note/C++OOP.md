只是个人学习过程中觉得一些值得注意的地方，并不是系统的笔记。

记笔记的唯一目的是加深印象，而不是方便查阅。事实上，个人觉得 C++ 内容庞大且复杂，而且重点似乎并不非常突出（因为处处是重点），如果为了方便查阅不妨备一本工具书。*《C++ Primer》*是一本非常不错的选择，它适合系统学习，也方便你对哪块内容模糊了回头翻阅，同时也是一本不错的参考书。

侯捷老师的课程，适合初学者对 C++ 面向对象部分不求甚解地大体一览，然后系统地学习 C++ 会更轻松；适合学习到一定程度后对 C++ 总体回味，还会收获颇丰。但如果想系统全面地认识、学习 C++ ，一本不错的书籍是更好的选择。

---

# 基于对象

## 不带指针的类 Complex

#### 关于头文件中的防卫式声明

在头文件中加入如下这样的防卫式声明是很有必要的，防止多次重复引入同一头文件：

```cpp
// complex.h
#ifndef __COMPLEX__
#define __COMPLEX__
...
#endif
```



#### 关于构造函数

写一个构造函数时，应尽量使用构造函数的特殊语法来初始化成员变量，而不要在函数体内采用赋值的方法。

```cpp
// 提倡的方法
complex (double r=0, double i=0)
  : re(r), im(i) {}

// 低效的方法
complex (double r=0, double i=0){
  re = r; im = i;
}
```



#### 关于常量成员函数

```cpp
// 在参数列表之后紧接一个 const 关键字，这样的成员函数称为常量成员函数
double real() const { return re; }
```

这里 const 的作用是修改隐式 this 指针的类型。在默认情况下 this 的类型是指向类类型的非常量版本的常量指针，因此对于没有 const 修饰的普通成员函数，将不能被一个常量对象调用（无法将指向非 const 对象的 this 绑定到一个 const 对象上）。

因此对于不应修改 data 的成员函数以 const 修饰，不止是为了防止使用者修改 data 而已。在一定程度上，不以 const 修饰是一种错误。

> 关于 隐式 this 指针，试着这样理解：对于一个 complex 对象 `c1` ，它调用 `real()` 成员函数时，则隐式地将一个指向对象 `c1` 的指针 `this` 传递给了函数 `real()` ，在函数内执行 `return this->re;` 从而取得对象 `c1` 的实部。
>
> 关于这一部分，可以参看 *《C++ Primer 中文版 第5版》* 第231页对 const 成员函数的介绍。



#### 关于友元

complex 类可以允许其他类或函数访问它的 private 成员，以 friend 关键词修饰对应的类或函数使其成为 complex 类的友元即可：

```cpp
class complex{
    // 将成员函数声明为友元
    friend complex& __doapl (complex*, const complex&);
    
    // 将非成员函数声明为友元
    friend complex complex& my_doapl (complex*, const complex&);
}
```

+ 相同 class 对各个对象互为友元（可以互相访问 private 成员）。



#### 关于重载函数

```cpp
// 以重载 complex 类的构造函数为例：
complex (double r=0, double i=0)
  : re(r), im(i) {}

complex () : re(0), im(0) {} // 不可行！
```

这样的重载函数是不可以的，也是多余的！虽然二者参数列表不同，但第一个构造函数为每一个形参都提供了默认值，当使用如下方法定义一个 complex 类时，编译器不知道该调用哪一个。

```cpp
// 两种方法等价，都表示不提供参数调用 complex 构造函数
complex c1;
complex c2();
```

<br>



## 带指针的类 String

#### 关于 Big Three

**class with pointer members** 必須要有 

+ **copy ctor** (拷贝构造) 
+ **copy op=** (赋值构造) 
+ **析构函数**



#### 关于拷贝构造函数

```cpp
inline
String::String(const String& str){
  	m_data = new char[ strlen(str.m_data) +1 ]; // 同类对象互为friend，直接取另一个object的private data
  	strcpy(m_data, str.m_data);
}


{
  	String s1("hello ");
  	String s2(s1);
//  String s2 = s1; 与s2(s1)一样调用拷贝构造函数，因为这里定义一个新对象，所以不是拷贝赋值
}
```



#### 关于拷贝赋值

```cpp
inline
String& String::operator=(const String& str){
  	if(this == &str) // 检测自我赋值
      	return *this;
  	
  	delete[] m_data; // 第一步
  	m_data = new char[ strlen(str.m_data) + 1 ]; // 第二步
  	strcpy(m_data, str.m_data); // 第三步
  	return *this;
}
```

**特别注意对自我赋值的检测！这一点尤其重要！**

看到的第一层似乎只是对效率的提升，而更深一层更为重要：如果没有对自我赋值的检测，那么进行第一步释放内存时即将自身的 data 给释放掉了，那第二步再要取 data 从何而来呢？