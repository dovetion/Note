# Java

[TOC]

## Part 1. 类与对象

### 对象内存分析

* java类属于引用数据类型，要进行内存管理
* 堆内存：保存的是对象的具体信息。栈内存：保存的是堆内存的地址，通过地址找到堆内存，再找到具体内容。程序中堆内存开辟是通过`new`完成的
* `Person per = new Person()`，`per`保存的是栈内存的名字，内容是堆空间中的对象的地址
* 对象实例化有两种方法：1.声明并实例化 2.分步操作

### 对象引用分析

* 同一块堆内存空间可以被不同栈内存所指向，也可以更换指向。

### 引用与垃圾

* “垃圾”，是没有任何栈内存指向的堆内存空间。所有的垃圾将被GC不定期回收，并且释放无用内存空间，但是如果垃圾过多，一定将影响到GC的处理性能，从而降低程序性能，程序开发中垃圾产生越少越好。

### 成员属性封装

* 采用封装性对属性进行保护（在类外部访问或更改属性）
* 使用`private`修饰属性，在类外部不可见，在内部可见
* 如果需要在类外访问封装属性，在Java开发标准中使用`setXxx()`和`getXxx()`方法，在类外调用相应方法。

### 构造方法与匿名对象

* 构造方法名称必须与类名称保持一致，不允许设置任何返回值类型，使用关键字`new`实例化对象的时候自动调用的
* 所有类中都有构造方法，如果没有写程序编译的时候自动创建。
* 没有返回值是为了和普通方法在结构上能使编译器区分，一个在实例化时调用一个在实例化后调用。
* 匿名对象：使用一次就成为垃圾。`new Person("gexin", 23).tell()`
* 只要是方法都可以传递任意数据类型（基本数据类型和引用数据类型）

### this关键字

* 程序里`this`可以实现三类结构的描述：
  * 当前类中的属性 
  * 当前类中的方法（普通方法和构造方法） 
  * 描述当前对象

* Java中，`{}`作为结构体的边界，内部有不去外面找，所以

  ```java
  public Person(String name, int a){
          name = name;
          age  = a;
  }
  ```

  会使name为`null`，因此属性上加上`this`访问

  ```
  public Person(String name, int a){
          this.name = name;
          this.age  = a;
  }
  ```

  在访问本类中的属性的时候，一定加上`this` 访问

* `this`也可以实现方法调用，但要考虑构造方法和普通方法

  * 构造方法调用`this()`：使用关键字`new`实例化对象的时候才会调用构造方法，在构造方法中执行。

  ```java
  public Person() {
      System.out.println("hi");
  }
  public Person(String name) {
      System.out.println("hi");
      this.name = name;
  }
  public Person(String name, int age){
      System.out.println("hi");
      this.name = name;
      this.age  = age;
  }
  ```

  可以优化成

  ```java
  public Person() {
      System.out.println("hi");
  }
  public Person(String name) {
      this();
      this.name = name;
  }
  public Person(String name, int age){
      this(name);
      this.age  = age;
  }
  ```

  对于本类构造方法的互相调用需要注意以下几点：

  1. 构造方法必须在实例化新对象的时候调用，`this()`只允许放在构造方法首行。
  2. 在普通方法中不可以调用构造方法，构造方法中可以调用普通方法。
  3. 构造方法必须有出口（避免递归调用构造方法）

  * 普通方法调用`this.method()`：实例化对象产生之后就可以调用普通方法

### static关键字

* `static`用来修饰属性

  * `static`定义属性（重复保存，修改不方便）：把属性修改成**公共属性**。`static`修饰的属性放在**全局数据区**，因此每一个对象不再拥有`static`属性，而指向全局数据区的属性。

  * static属性可以由**类**名称直接调用，也可以在没有对象实例化的时候使用。

* `static`修饰方法

  * `static`方法可以由类名直接调用
  * **`static`方法只可以调用`static`属性或者`static`方法**，非`static`方法可以调用`static`属性和`static`方法，因为`static`的属性和方法可以在没有实例化对象的情况下调用。（所以static方法中不能使用`this`）
  * `static`定义的方法或者是属性不是代码编写之初要考虑的内容，只有在回避实例化或描述公共属性的情况下才考虑static定义的方法或者是属性。



### 代码块

在程序中，使用`{}`定义的结构就叫做代码块。根据代码块出现的位置和关键字分为“普通代码块”，“构造块”，“静态块”和“同步块”（多线程）。

* **普通代码块**是定义在一个方法中的代码块。按Java标准，相同名称的变量不能在同一个代码块中存在。可以在方法中进行结构拆分，防止相同变量名带来的互相影响。

  ```java
  public void method() {
  	{
  		int x = 10;
  	}
  	int x = 100;
  }
  ```

  

* **构造代码块**定在类中，构造块会优先于构造方法执行，并且每一次实例化新对象的时候都会调用构造块中的代码

  ```java
  class Person{
  	public Person(){
  		System.out.println("构造方法执行")
  	}
  	{
  		System.out.println("构造块执行")
  	}
  }
  
  //output:
  //构造块执行
  //构造方法执行
  ```

* **静态代码块**，考虑两种情况，主类中定义静态块，非主类中定义静态块

  ```java
  class Person{
  	public Person(){
  		System.out.println("构造方法执行")
  	}
  	{
  		System.out.println("构造块执行")
  	}
      static {
      	System.out.println("静态块执行")
      }
  }
  //output:
  //静态块执行
  //构造块执行
  //构造方法执行
  ```

  

  * 非主类中定义静态块，主要是为了**静态属性的初始化，并且只会执行一次**
  * 主类中定义静态块，**静态代码块优先于主方法执行**



---



## Part 2. 基本操作

### 数组的定义与使用

数组：一组相关变量的集合，数组为**引用数据类型**，用`new`分配空间

* 定义格式`类型[] 名称 = new 类型[长度]`，`类型 名称[] = new 类型[长度]`

* 数组静态初始化：在数组定义的时候设置好了内容

  * 简化格式： `类型[] 名称 = {data1, data2, ...}`

  * 完整格式： ` 类型[] 名称 = new 类型[]{data1, data2, ... }`

### 数组引用传递

一个堆内存可以背多个栈内存所指向。

```java
int[] data = new int[]{10, 20, 30};
int[] tmp = data;
tmp[0] = 99;

//result:{99, 10, 20}
```



### foreach输出

jdk1.5后为了减轻下标对程序的影响，将数组中每一个元素取出保存在变量里。**对变量的操作不影响数组内容**

```java
int[] data = new int[]{1, 2, 3};
for(int x: data){
	...
}
```

###  数组相关类库

* `java.util.Arrays`提供各种方法比如：`java.util.Arrays.sort(data)`
* 数组拷贝 `System.arraycopy(原数组, 原数组开始点, 目标数组, 目标数组开始点, 拷贝长度)`

### 方法可变参数

从JDK1.5，对方法参数有新的支持，采用可变参数

```java
public static int sum(int ... data) {
	int sum = 0;
	for (int tmp: data)
		sum += tmp;
	return sum; 
}
```

### 引用传递实际应用

引用传递是Java开发中最重要的技术组成。

* 类的关联：“人”有“车”，“车”有“人”（车主）。类中相互引用。

  ```java
  class Person {
  	private Car car;
  	public void setCar(Car car){
  		this.car = car;
  	}
  }
  class Car {
  	private Person person
  	public void setPerson(Person per){
  		this.Person = per;
  	}
  }
  ```

* 自身关联

  ```java
  class Person{ 
  	private Person[] children;
  	public void setChildren(Person[] children){
  		this.children = children;
  	}
      public Person[] getChildren(){
      	return this.children;
      }
  }
  ```

这些都是通过引用数据类型的关联来完成的



---



## Part 3. String类

