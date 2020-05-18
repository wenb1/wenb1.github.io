---
layout: post
title: 设计模式之原型模式(Prototype Pattern)
categories: DesignPattern
description: 原型模式介绍
keywords: DesignPattern, 设计模式
---

原型模式(Prototype Pattern)用于创建重复的对象，同时又能保证性能。它属于创建型设计模式，它提供了一种创建对象的最佳方法。使用原型实例指定将要创建的对象类型，通过复制这个实例创建新的对象。

另外在软件系统中，有些对象的创建过程较为复杂，而且有时候需要频繁创建，原型模式通过给出一个原型对象来指明所要创建的对象的类型，然后用复制这个原型对象的办法创建出更多同类型的对象，这就是原型模式的意图所在。

# 1. 原型模式与拷贝

在原型模式结构中定义了一个抽象原型类，所有的Java类都继承自`java.lang.Object`，而`Object`类提供一个`clone()`方法，可以将一个Java对象复制一份。因此在Java中可以直接使用`Object`提供的`clone()`方法来实现对象的克隆，Java语言中的原型模式实现很简单。

能够实现克隆的Java类必须实现一个标识接口`Cloneable`，表示这个Java类支持复制。如果一个类没有实现这个接口但是调用了`clone()`方法，Java编译器将抛出一个`CloneNotSupportedException`异常。`java.lang.Cloneable` 只是起到告诉程序可以调用`clone()`方法的作用，它本身并没有定义任何方法。

在使用原型模式克隆对象时，根据其成员对象是否也克隆，原型模式可以分为两种形式：**深拷贝**和**浅拷贝**。我们有必要先了解浅拷贝和深拷贝来理解原型模式如何克隆对象。

## 1.1 Java中对象的创建

在Java中，我们有两种方式创建对象：

1. 使用`new`关键字创建一个对象
2. 使用`clone()`方法

`new`操作符的本意是分配内存。程序执行到new操作符时， 首先去看`new`操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。

`clone()`在第一步是和`new`相似的， 都是分配内存，调用`clone()`方法时，分配的内存和源对象（即调用`clone()`方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，`clone()`方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部*。

## 1.2 复制对象和复制引用

### 1.2.1 复制引用

```java
public class PrototypeDemo {
	
	public static void main(String[] args) {
		Person person1 = new Person();
		Person person2 = person1;
		System.out.println(person1);
		System.out.println(person2);
	}
	
}
/*output:
 * dev.wenbo.designpattern.prototype.Person@15db9742
 * dev.wenbo.designpattern.prototype.Person@15db9742
 */

```

可以看到两个引用打印出的对象地址是一样的，说明这两个引用实际上是引用的同一个对象。我们实际上只复制了引用。

### 1.2.2 复制对象

```java
public class PrototypeDemo {
	
	public static void main(String[] args) throws CloneNotSupportedException {
		Person person1 = new Person();
		Person person2 = (Person) person1.clone(); 
		System.out.println(person1);
		System.out.println(person2);
	}
	
}
/*output:
 * dev.wenbo.designpattern.prototype.Person@15db9742
 * dev.wenbo.designpattern.prototype.Person@6d06d69c
 */
```

打印之后的结果显示出这两个对象的存储地址不同，所以实际创建出了两个对象，我们真正复制出了一个对象。

## 1.3 深拷贝和浅拷贝

我们定义一个`Person`类，有两个属性`age`和`gender`，分别是`int`类型和`String`类型。

```java
public class Person implements Cloneable{
	private int age;
	private String gender;
	
	
	public Person() {
	}


	public Person(int age, String gender) {
		this.age = age;
		this.gender = gender;
	}


	public int getAge() {
		return age;
	}


	public void setAge(int age) {
		this.age = age;
	}


	public String getGender() {
		return gender;
	}


	public void setGender(String gender) {
		this.gender = gender;
	}
	
	protected Object clone() throws CloneNotSupportedException {
		return (Person) super.clone();
	}
}
```

由于age是基本数据类型， 那么对它的拷贝没有什么疑议，直接将一个4字节的整数值拷贝过来就行。但是name是String类型的， 它只是一个引用， 指向一个真正的String对象，那么对它的拷贝有两种方式： **①直接将源对象中的name的引用值拷贝给新对象的name字段，也就是我们前面提到的复制引用** **②根据原Person对象中的name指向的字符串对象创建一个新的相同的字符串对象，将这个新字符串对象的引用赋给新拷贝的Person对象的name字段，也就是复制对象。** 这两种拷贝方式分别叫做 **浅拷贝** 和 **深拷贝** 。

我们可以通过比较复制对象和源对象的`gender`属性的地址值是否相同来验证`clone()`方法是深拷贝还是浅拷贝：

```java
public class PrototypeDemo {
	
	public static void main(String[] args) throws CloneNotSupportedException {
		Person person1 = new Person(23, "male");
		Person person2 = (Person) person1.clone(); 
		System.out.println(person1.getGender() == person2.getGender());
	}
	
}
/*output:
 * true
 */
```

我们得到`true`，也就说明`gender`属性引用的是同一对象，说明`clone()`实际是浅拷贝。

如果我们想让`clone()`方法进行深拷贝，可以覆写`Object`类的`clone()`方法，除了调用父类中的`clone()`方法得到新的对象， 还要将该类中的引用变量也clone出来。如果只是用`Object`中默认的`clone()`方法，是浅拷贝的。

我们重新定义一个`Person`类：

```java
public class Person implements Cloneable{
	private int age;
	private Job job;
	
	public Person() {
	}

	public Person(int age, Job job) {
		this.age = age;
		this.job = job;
	}


	public int getAge() {
		return age;
	}


	public void setAge(int age) {
		this.age = age;
	}
	
	public Job getJob() {
		return job;
	}


	public void setJob(Job job) {
		this.job = job;
	}
	
	protected Object clone() throws CloneNotSupportedException {
		Person newPerson = (Person) super.clone();
        //对属性job也进行克隆
		newPerson.job = (Job) job.clone();
		return newPerson;
	}
}

public class Job implements Cloneable{
	
	@Override
	protected Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}
```

进行测试：

```java
public class PrototypeDemo {
	
	public static void main(String[] args) throws CloneNotSupportedException {
		Person person1 = new Person(23, new Job());
		Person person2 = (Person) person1.clone(); 
		System.out.println(person1 == person2);
		System.out.println(person1.getJob() == person2.getJob());
	}
	
}
/*output:
 * false
 * false
 */
```

这次我们看到，`Person`里对其属性`job`也进行了克隆，结果就是进行了深拷贝。

那么再深入一步，如果我们的`Person`类中的`Job`类属性中还有其他引用属性呢？比如`Job`中还有`salary`，`workplace`等属性引用，我们是不是还要再在`Job`类中克隆`salary`，`workplace`等属性呢？

其实，对于`Person`类的对象来说，已经算是深拷贝了，因为`Person`对象内引用的对象都进行了拷贝，也就是说两个`Person`对象内的属性都引用两个独立的对象。应该说，这是一种**不彻底的深拷贝**。

要想进行彻底的深拷贝，就要把所有属性引用的对象的引用对象都克隆一份，也就是说，实际是一层套一层的，这才算是彻底的深拷贝。对应我们这这个例子，就是假设，`Job`类中还有`salary`，`workplace`等属性的话，要把它们也都克隆一份，以此类推。

# 2. 原型模式实现

## 2.1 创建一个实现了`Cloneable`接口的抽象类 Shape

```java
public abstract class Shape implements Cloneable {
   
   private String id;
   protected String type;
   
   abstract void draw();
   
   public String getType(){
      return type;
   }
   
   public String getId() {
      return id;
   }
   
   public void setId(String id) {
      this.id = id;
   }
   
   public Object clone() {
      Object clone = null;
      try {
         clone = super.clone();
      } catch (CloneNotSupportedException e) {
         e.printStackTrace();
      }
      return clone;
   }
}
```

## 2.2 创建扩展抽象类的实体类

```java
//Rectangle
public class Rectangle extends Shape {
 
   public Rectangle(){
     type = "Rectangle";
   }
 
   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}

//Square
public class Square extends Shape {
 
   public Square(){
     type = "Square";
   }
 
   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}

//Circle
public class Circle extends Shape {
 
   public Circle(){
     type = "Circle";
   }
 
   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

## 2.3 创建一个类，从数据库获取实体类，并把它们存储在一个`Hashtable`中

```java
public class ShapeCache {
    
   private static Hashtable<String, Shape> shapeMap 
      = new Hashtable<String, Shape>();
 
   public static Shape getShape(String shapeId) {
      Shape cachedShape = shapeMap.get(shapeId);
      return (Shape) cachedShape.clone();
   }
 
   // 对每种形状都运行数据库查询，并创建该形状
   // shapeMap.put(shapeKey, shape);
   // 例如，我们要添加三种形状
   public static void loadCache() {
      Circle circle = new Circle();
      circle.setId("1");
      shapeMap.put(circle.getId(),circle);
 
      Square square = new Square();
      square.setId("2");
      shapeMap.put(square.getId(),square);
 
      Rectangle rectangle = new Rectangle();
      rectangle.setId("3");
      shapeMap.put(rectangle.getId(),rectangle);
   }
}
```

## 2.4 测试

```java
public class PrototypeDemo {
	
	public static void main(String[] args) {
	      ShapeCache.loadCache();
	 
	      Shape clonedShape = (Shape) ShapeCache.getShape("1");
	      System.out.println("Shape : " + clonedShape.getType());        
	 
	      Shape clonedShape2 = (Shape) ShapeCache.getShape("2");
	      System.out.println("Shape : " + clonedShape2.getType());        
	 
	      Shape clonedShape3 = (Shape) ShapeCache.getShape("3");
	      System.out.println("Shape : " + clonedShape3.getType());        
	   }
	
}
```

# 3. 原型模式分析

## 3.1 优点

- 当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，通过一个已有实例可以提高新实例的创建效率。
- 可以动态增加或减少产品类。
- 原型模式提供了简化的创建结构。
- 可以使用深克隆的方式保存对象的状态。

## 3.1 缺点

- 需要为每一个类配备一个克隆方法，而且这个克隆方法需要对类的功能进行通盘考虑，这对全新的类来说不是很难，但对已有的类进行改造时，不一定是件容易的事，必须修改其源代码，违背了“开闭原则”。
- 在实现深克隆时需要编写较为复杂的代码。

## 3.1 适用场景

- 资源优化场景。

- 类初始化需要消耗非常多的资源，这个资源包括数据、硬件资源等。

- 性能和安全要求的场景。

- 通过new产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。

- 一个对象多个修改者的场景。

- 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。

- 在实际项目中，原型模式很少单独出现，一般是和工厂模式一起出现，通过`clone`方法创建一个对象，然后由工厂方法提供给调用者。

**参考文章**

[详解Java中的clone方法 -- 原型模式 (昨夜星辰_zhangjg)](https://blog.csdn.net/zhangjg_blog/article/details/18369201/)

[Java设计模式（四）之原型模式 (12313凯皇)](https://www.jianshu.com/p/f5b17caa6d66)

[深入理解原型模式 ——通过复制生成实例 (Guide哥)](https://blog.csdn.net/qq_34337272/article/details/80706444)