---
layout: post
title: 梳理一些C++里比较难记的语法特性
category: 高大上然并卵
tags: [ 'c++', 'programming' ]
---

最近在重新学习C++，有一些特性之前就看到过很多遍但总是记不住。我想大概光靠看书能吸收的知识还是太有限了，所以在这里把一些特性重新梳理一遍。

## virtual function 和 override

在override这个事情上，C++和大部分其他语言不同。子类和父类拥有同名函数的时候，并不会默认子类重写父类的函数。只有当父类的函数声明为virtual时，重写才会发生。而当父类的函数没有声明为virtual时，拥有父类类型的子类会调用父类的方法。（详见下面的例子）

```C++
struct B {
  virtual void f() const {cout << "B::f ";}
  void g() const {cout << "B::g ";}
};

struct D : D {
  void f() const {cout << "D::f ";}
  void g() const {cout << "D::g ";}
};

struct DD : D {
  void f() const {cout << "DD::f ";}
  void g() const {cout << "DD::g ";}
};

void call(const B& b) {
  b.f();
  b.g();
}

int main() {
  B b;
  D d;
  DD dd;

  call(b); // B::f B::g
  call(d); // D::f B::g
  call(dd); // D::f B::g

  b.f(); // B::f
  b.g(); // B::g

  d.f(); // D::f
  d.g(); // D::g

  dd.f(); // DD::f
  dd.g(); // DD::g
}
```
注意`call(dd)`这一句，输出的结果是`D::f`而不是`DD::f`。

### 利用编译器防止意外未重写

可以在函数声明里加入`override`来声明这个函数是要重写父类函数的，如果它没有重写某个函数的话，编译器就会报错：

```C++
struct B {
  virtual void f() const {cout << "B::f ";}
  void g() const {cout << "B::g ";}
};

struct D : D {
  void f() override {cout << "D::f ";} // ERROR!!!
  void g() const {cout << "D::g ";}
};
```
在上面的例子里`D::f`忘记声明为`const`，导致其未重写任何函数，编译器会报错。

## Access Control for Base Class

一个类中的变量和方法可以声明为`public`，`protected`, `private`这个事情大家都懂，在C++里比较麻烦的是在声明继承时，父类同样可以声明为`public`，`protected`或者`private`。它的含义是从父类的中继承过来的方法和变量可以在哪里使用，这个可以理解为在继承的过程中，对父类的方法和变量的权限进行了一次重新的定义。需要注意的事这种重新定义只允许让权限更严格，而不能更宽松（显然应该是这样）。如果不声明父类权限的话，父类权限默认为`private`，但这通常都不是我们想要的行为。一般情况下，我们都会把父类定义为`public`以让父类中的方法和以前具有相同的权限。

需要注意的是，以上说的已经假定子类定义为`class`。如果子类定义为`struct`，那它的父类权限默认为`public`。在C++里，`struct`就等于方法和变量默认为`public`的`class`，除此之外没有任何区别。而`class`一切都默认为`private`。所以在上面的例子里，`B`，`D`，`DD`都定义为`struct`就省去了写很多`public`的麻烦。究竟是使用`struct`更好还是使用`public`更好，这种争论大概就和大括号是否要换行一样，虽然各有各的道理，但其实是完全等价的。下面给一个例子：

```C++
class Circle : public Shape {public: /* ... */}; // OK
struct Circle : Shape {/* ... */}; // Completely equivalent to previous one
class Circle : Shape {public: /* ... */}; // probably a mistake: all methods in Shape cannot be called in Circle
```

## Copy Constructors and Copy Assignment

关于这个话题，最主要的一点是要清楚在什么时候会调用什么东西，因为调用的形式和声明的形式往往很不一样：

```C++
SomeType a;
SomeType b = a; // This will call the copy constructor of SomeType
SomeType c;
c = a // This will call the copy assignment of SomeType
```
