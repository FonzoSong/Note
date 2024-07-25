

#### virtual function
在基类中声明,在派生类中重写的函数;
- 确保调用正确的函数;而不管用于函数调用的引用或指针的类型如何.
- 在基类中使用 `virtual` 声明

1. 虚拟函数不能是静态的。
2. 虚函数可以是另一个类的友函数。
3. 应使用基类类型的指针或引用访问虚拟函数，以实现运行时多态性。
4. 虚拟函数的原型在基类和派生类中应该是相同的。
5. 它们始终在基类中定义，并在派生类中重写。派生类不强制覆盖（或重新定义虚函数），在这种情况下，使用函数的基类版本。
6. 类可以具有虚拟析构函数，但不能具有虚拟构造函数。

## **虚拟功能的工作（VTABLE和VPTR的概念）**

1. 如果创建了该类的对象，则会插入**一个虚拟指针 （VPTR）** 作为该类的数据成员，以指向该类的 VTABLE。对于创建的每个新对象，都会插入一个新的虚拟指针作为该类的数据成员。
2. 无论是否创建对象，该类都包含**一个名为 VTABLE 的函数指针的静态数组**作为成员。此表的单元格存储该类中包含的每个虚函数的地址。