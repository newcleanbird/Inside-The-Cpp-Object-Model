# class 和 struct
## 关键词所带来的差异(A Keyword Distinction )
struct的默认访问修饰符是public；而class的默认访问修饰符是private。
struct的默认继承方式是public，而class的默认继承方式是private。除此之外使用时没有区别。
- 某种意义上, 在C++中struct和class这两个关键字是可以互换的。

### 关键字的困扰：struct 和 class
- class不仅是一个关键字, 它还会引入它所支持的封装和继承的哲学。
- 为了维护c++与C语言之间的兼容性，重载函数的解决方式变得很复杂。

### 策略性正确的struct（The Politically Correct Struct）
这里讲述的是在C中，struct没有访问修饰符，因此struct中的成员变量是按声明顺序出现的内存中的，而在C++中，class/struct有访问修饰符，因此成员变量未必是按照声明顺序出现在内存中的。
- 组合而非继承, 才是把C和C++结合在一起的唯一可行方法。