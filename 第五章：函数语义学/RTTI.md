# RTTI（Runtime Type Identification，运行时类型识别）
RTTI（Runtime Type Identification，运行时类型识别）是C++语言中的一项机制，允许程序在运行时动态地确定对象的类型。这项机制对于实现诸如类型安全的向下转型（downcasting）、实现基于类型的行为以及调试等场景非常有用。RTTI主要通过两个关键特性实现：``typeid``操作符和``dynamic_cast``操作符。

## typeid操作符
typeid是一个C++关键字，用于在编译时或运行时获取表达式或类型的名字。当应用于一个对象时，它会返回一个包含该对象实际类型信息的std::type_info对象的引用。std::type_info对象包含了类型的名字（通常为人类可读的字符串形式）以及其他可能的类型相关信息，可以用来比较类型是否相同。

使用typeid的一个典型场景是确定对象的动态类型，尤其是在多态环境中。例如，当你有一个指向基类的指针，但实际上它指向的是一个派生类对象，typeid可以帮助你识别出这一点。

## dynamic_cast操作符
dynamic_cast是另一个C++提供的运行时类型转换操作符，它允许安全地将基类的指针或引用转换为派生类的指针或引用。与C风格的强制类型转换（如reinterpret_cast或旧式的C样式cast）不同，dynamic_cast会在转换前检查转换的安全性。如果转换是合法的（即对象实际上是目标派生类的实例），则转换成功；否则，dynamic_cast会返回一个空指针（对于指针类型）或抛出一个std::bad_cast异常（对于引用类型，仅在使用RTTI时）。

dynamic_cast依赖于对象的类型信息，因此只有当转换的源类型含有至少一个虚函数（从而具有虚函数表）时，dynamic_cast才能工作。这是因为虚函数的存在表明了类型系统中存在多态性，而这是dynamic_cast实现其功能的基础。

## RTTI的启用与禁用
RTTI默认在大多数编译器中是启用的，但也可以通过编译器选项禁用，以减少代码体积或提高性能，尤其是在不需要类型识别的环境下。禁用RTTI意味着typeid和dynamic_cast将无法正常工作。

## 何时使用RTTI
尽管RTTI提供了强大的动态类型检查和转换能力，但过度依赖它可能导致代码的结构不够清晰，并可能降低效率。设计良好的面向对象程序往往更倾向于使用虚函数来实现多态行为，减少对RTTI的直接需求。然而，在某些特定情况下，如在日志记录、序列化、或需要进行复杂的类型检查和转换的框架中，RTTI仍是一个不可或缺的工具。

## RTTI例子
```cpp
	Base *pb = new Derive(); // 父类指针指向一个子类对象
	pb->g();

	Derive myderive;
	Base &yb = myderive;	// 父类引用一个子类对象
	yb.g();
```
1. c++运行时类型识别RTTI，要求父类中必须至少有一个虚函数；如果父类中没有虚函数，那么得到RTTI就不准确；
2. RTTI就可以在执行期间查询一个多态指针，或者多态引用的信息了；
3. RTTI能力靠typeid和dynamic_cast运算符来体现；
```cpp
	cout << typeid(*pb).name() << endl;	// class derive
	cout << typeid(yb).name() << endl;	// class derive

	Derive *pderive = dynamic_cast<Derive *>(pb);
	if (pderive != NULL)
	{
		cout << "pb实际是一个Derive类型" << endl;
		pderive->myselffunc(); //调用自己专属函数
	}	
```

C++引入RTTI的目的：让程序在运行时，通过基类指针或者基类引用来获得这个指针或者引用的所指向的对象的类型。

### RTTI实现原理
1. typeid返回的是一个常量对象的引用，这个常量对象的类型一般是 type_info（类）；
```cpp
const std::type_info &tp = typeid(*pb);
	Base *pb2 = new Derive();
	Base *pb3 = new Derive();
	const std::type_info &tp2 = typeid(*pb2);
	const std::type_info &tp3 = typeid(*pb3);

	cout << typeid(int).name();

	if (tp == tp2)
	{
		cout << "很好，类型相同" << endl;
	}
```
2. typeid的其他用法
```cpp
// 静态类型：
	cout << typeid(int).name() << endl;
	cout << typeid(Base).name() << endl;
	cout << typeid(Derive).name() << endl;
	Derive *pa3 = new Derive();
	cout << typeid(pa3).name();

// 多态
	Base *pb = new Derive();
	Derive myderive;
	Base &yb = myderive;
	cout << typeid(*pb).name() << endl; //class Derive
	cout << typeid(yb).name() << endl; //class Derive
	Base *pb2 = new Derive();
	const std::type_info &tp2 = typeid(*pb2);
```
总结：RTTI的测试能力跟基类中是否u才能在虚函数表有关系，如果基类中没有虚函数，也就不存在基类的虚函数表，RTTI就无法得到正确结果；


3. 获取RTTI与虚函数表有直接关系。
	虚函数表指针vptr --> 虚函数表首地址
	虚函数表中依次保存虚函数地址
	RTTI相关地址 保存在在虚函数表首地址前一个地址 --> 指向一个内存空间的首地址 + 12字节(与RTTI有关的结构信息)之后 保存的是 type_info对象的首地址 --> 对象type_info结构信息
```cpp
	printf("tp2地址为:%p\n", &tp2); // 实际上的type_info的结构信息地址(目标地址)

	long *pvptr = (long *)pb2;		// 将pb2的指针转为 long* 方便计算
	long *vptr = (long *)(*pvptr);	// 取pb2所指向的对象的首地址，即虚函数表指针vptr指向的地址。
	printf("虚函数表首地址为:%p\n", vptr);
	printf("虚函数表首地址之前一个地址为:%p\n", vptr-1); //这里的-1实际上是往上走了4个字节

	long *prttiinfo = (long *)(*(vptr - 1));
	prttiinfo += 3; //跳过12字节
	long * ptypeinfoaddr = (long *)(*prttiinfo);
	const std::type_info *ptypeinfoaddrreal = (const std::type_info *)ptypeinfoaddr;
	printf("ptypeinfoaddrreal地址为:%p\n", ptypeinfoaddrreal);
	cout << ptypeinfoaddrreal->name() << endl; 
```

### 三：vptr,vtbl,rtti的type_info信息 构造时机
	rtti的type_info信息:编译后就存在了；写到了可执行文件中
	总结一下：各个编译器实现有一定差异，但总体都是以虚函数表开始地址为突破口；