# 嵌入式C++用法集锦

## 单例写法

```
static x& GetInstance()
{
  static x instance; // x为类名
  return instance;
}
x(const &x) = delete;
x& operator = (const &x) = delete;
```
- 为什么要用static
  
  - 如果该函数不是静态，即作为普通的成员函数，需要先创建实例再调用，显然违背单例设计原则。static使得函数属于类本身。
  - 保证内存唯一。静态局部变量在程序生命周期内只会被初始化一次。无论调用多少次 GetInstance()，返回的永远是内存中同一个地址上的那个对象。
  - 使用了`static`后，`instance`对象会被存储在全局/静态存储区。它的生命周期从第一次执行一直持续到程序完全结束。

- 为什么不写成`static x* instance`静态成员指针
  
  这是最老牌的 C++ 单例写法。需要在类内声明 `static HostPC* instance;`，然后在 `.cpp` 文件里初始化 `HostPC* HostPC::instance = nullptr;`，并在 `GetInstance` 里判断 `if (instance == nullptr) { instance = new HostPC(); }`。最后需要手动`delete`，否则会内存泄漏。并且，在多线程环境下，两个线程可能同时判断 `instance == nullptr`，导致创建了两个对象（竞态条件）。要解决这个问题，必须加锁（Mutex），代码会变得非常臃肿。
  
- 为什么不写成`static x& instance`静态成员引用
  - 引用：变量的别名。必须初始化；不可变更绑定其他变量（和指针的本质区别）；不可为空，必须指向合法的内存
  - ```
    int a = 10;
    int& ref = a; // ref 是 a 的引用
    ref = 20;    // 修改 ref，a 也会变成 20
    // int& ref是错误写法
    ```
- 禁用拷贝构造函数
  
  `x(const &x) = delete;`
  
  防止写 `x another_instance = x::GetInstance();`，这会拷贝出单例的一个副本。以串口业务为例，如果析构函数里有关闭串口的操作，其中一个对象销毁时把串口关了，另一个对象就会直接崩溃。
  
- 禁用赋值操作符

  `x& operator = (const x&) = delete;`

  防止先创建一个对象，然后赋值给他，违背单例设计原则
