[TOC]

## 实现

### 尽可能延后变量定义式的出现时间

- 只要定义了一个变量而其类型带有一个构造函数或析构函数，那么当程序的控制流到达这个变量的定义式时，就会产生构造成本；当这个变量离开作用域时，就会产生析构成本，即使这个变量未被使用；
- 不知应该延后变量的定义，直到非得使用该变量的那一刻为止，甚至应该尝试延后这份定义直到能够给他初值为止，这样 **能够避免构造（或析构）非必要对象，还可以避免无意义的默认构造行为**。

### 尽量少做转型操作

- `const_cast<T>()`、`dynamic_cast<T>()`、`reinterpret_cast<T>()` 和 `static_cast<T>()`；

### 避免返回 handles 指向对象内部成分

- 引用、指针、迭代器统统可称为 handles（号码牌，用来取得某个对象）；而返回一个代表数据内部的 handle，会降低对象的封装性；

- 成员变量的封装性最多只等于返回其引用的函数的访问级别；绝不应该令成员函数返回一个指针指向 “访问级别较低” 的成员函数。

### 为异常安全努力是值得的

- 异常安全的两个条件：**不泄露任何资源** 和 **不允许数据败坏**；
- 解决资源泄露的问题可以通过对象管理资源实现；
- 

## 继承与面向对象设计

### 区分接口继承与实现继承

- 接口继承和实现继承不同。在公有继承下，派生类总是继承基类的接口；
- 纯虚函数只具体指定接口继承；
- 非纯虚函数具体指定接口继承及缺省实现继承；
- 普通非虚函数具体指定接口继承以及强制性实现继承；

### 虚函数外的其他选择

- **Non-Virtual Interface**：以公有的非虚成员函数包裹较低访问性（private 或 protect）的虚函数；
- 将虚函数替换为 **函数指针成员变量**；
- 将继承体系内的虚函数替换为另一个继承体系内的虚函数；

### 绝不重新定义继承来的非虚函数

- 对象的静态绑定类型是其在程序中 **被声明时** 所采用的类型；对象的动态绑定类型则是指 **目前所指对象的类型**；

- 类的非虚成员函数的调用是 **编译时静态绑定**；虚成员函数则是 **运行时动态绑定**；

### 绝不重新定义继承而来的缺省值

- 虚函数是动态绑定，而缺省参数却是 **静态绑定**；
- C++坚持这种方式是考虑到 **运行期效率**；如果缺省参数值是动态绑定的，编译器就必须有某种方法在运行期为虚函数决定适当的参数缺省值，这比目前的编译期决定机制更慢更复杂；

### 明智而审慎地使用 private 继承

- 如果类之间的继承关系是 **私有的 private**，编译器不会自动将一个派生类对象转换为一个基类对象；
- 由基类私有继承来的所有成员，在派生类中都会变成私有属性；
- 私有继承意味着 **根据某物实现出**，其纯粹只是一种实现技术；
- 尽可能使用复合，必要时才使用私有继承；
- 私有继承主要用于 **当派生类需要访问基类的保护成员，或需要重新定义继承来的一个或多个虚函数**；
- 有一种激进情况涉及空间最优化，也会促使使用私有继承：
  - C++ 裁定凡是 **独立非附属的对象都必须具有非零大小**，即空类也会有大小；
  - C++ 会默默安插一个 `char` 到空对象内；
  - 上述约束不适用于派生类对象内的基类对象成分，因为它们并不是独立非附属；
  - 如果私有继承一个空类，空类就不会有大小，这叫做 **空白基类最优化 EBO**.

### 明智而审慎地使用多重继承

- 多重继承比单一继承复杂，可能导致新的歧义性，以及对 **虚继承** 的需求；
- 虚继承会增加大小、速度、初始化（赋值）复杂度等成本。如果虚基类不带任何数据，将是最具有实用价值的情况；
- 多重继承的正当用途：公有继承某个接口类，私有继承某个协助实现的类。

## 模板与泛型编程

### 了解隐式接口和编译期多态

- 面向对象编程总是以 **显示接口** 和 **运行期多态** 解决问题；模板及泛型编程中，**隐式接口** 和 **编译期多态** 重要性上升；
- **以不同的模板参数具现化函数模板会导致调用不同的函数**，这便是编译期多态；
- 运行期多态和编译期多态之间的差异，类似于 “哪一个重载函数该被调用”（发生在编译期）和 “哪一个虚函数该被绑定“（发生在运行期）之间的差异；
- 通常，显示接口由函数的 **签名式（函数名称、参数类型、返回类型）**构成；
- 隐式接口由 **有效表达式** 组成；

### 了解 typename 的双重意义

- 从 C++ 的角度来看，声明模板参数时，不论使用关键字  `class` 或 `typename`，意义完全相同；
- 模板内出现的名称如果相依于某个模板参数，称之为 **从属名称**；如果从属名称在 class 内呈嵌套状，称之为 **嵌套从属名称**；
- 如果解析器在模板中遭遇一个嵌套从属名称，他便假设这个名称不是个类型，除非你告诉他是；
- 因此，任何时候当你想要在模板中指涉一个嵌套从属类型名称，就必须在紧邻它的前一个位置放上关键字 **typename**；
- **typename** 只被用来验明嵌套从属类型名称，其它名称不该使用；
- 但不得在基类列表或者成员初始化列表内将其作为基类修饰符；

### 学习处理模板化基类内的名称

- 在模板C++中，C++不会进入模板化基类观察，即拒绝在模板化基类中寻找继承而来的名称，因为编译器知道基类模板有可能会被特化，而那个特化的版本可能不提供和一般性模板相同的接口；

- 避免这一特性的三种方法：

  1. 在基类函数调用动作之前加上 **this->**，假设成员函数将被继承；
  2. 使用 **using 声明式**，告诉编译器，请它假设成员函数位于基类中；
  3. 明白地指出被调用函数位于基类中，但如果被调用函数是虚函数，这样的行为会关闭虚函数绑定行为；

  ``` c++
  MsgSender<Company>::sendClear(info);
  ```

### 将与参数无关的代码抽离模板

- 

### 运用成员函数模板接收所有兼容类型

- **同一个模板的不同具现体之间并不存在什么与生俱来的固有关系**；例如，如果以带有 ”基类-派生“ 关系的两类型分别具现化某个模板，产生出来的两个具现体之间并不带有 ”基类-派生“ 关系
- 可以通过 **成员函数模板** 为模板类写一个 **构造模板**：

``` c++
template<typename T>
class SmartPtr {
public:
    template<typename U>
    SmartPtr(const SmartPtr<U> &other);
};
```

- 对任何类型 T 和任何类型 U，可以根据 SmartPtr<U\> 生成一个 SmartPtr<T\>，这一类构造函数叫作 **泛化拷贝构造函数**。上面的泛化拷贝构造函数并未被声明为 explicit，因为原始指针类型之间的转换是隐式转换；
- 成员函数模板的效用并不局限于构造函数，常扮演的另一个角色是 **赋值操作**；
- 成员函数模板并不会改变语言规则，即 **在类内声明泛化拷贝构造函数并不会阻止编译器生成它们自己的默认拷贝构造函数（一个非模板函数）**；如果想要控制拷贝构造的方方面面，必须同时声明泛化拷贝构造函数和正常的拷贝构造函数；赋值操作亦如此；

### 需要类型转换时请为模板定义非成员函数

- 在模板实参推断过程中从不将隐式类型转换函数纳入考虑；
- **模板类中的友元声明式可以指涉某个特定函数**；因为 **模板类并不依赖于模板实参推断（模板实参推断只用于函数模板）**，所以编译器总是能够在模板具现化时得到模板类型 T，同时，友元函数也被实例化得到，成为一个函数而非函数模板；