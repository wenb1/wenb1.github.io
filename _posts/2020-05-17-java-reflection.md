---
layout: post
title: Java之反射(Reflection)
categories: Java
description: 讲解Java中的反射
keywords: Java, reflection, 反射
---

Java反射机制(Reflection)是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为Java语言的反射机制。

# 1. 反射基本介绍

简单来说，反射就是把Java类中的各个组成部分，比如成员变量，构造方法，方法等信息，映射成Java对象，有了这些对象后就可以做其它事。

一般来说反射是用来做框架的，或者说可以做一些抽象度比较高的底层代码。在日常的第三方应用开发过程中，经常会遇到某个类的某个成员变量、方法或是属性是私有的或是只对系统应用开放，这时候就可以利用Java的反射机制通过反射来获取所需的私有成员或是方法。当然，也不是所有的都适合反射，之前就遇到一个案例，通过反射得到的结果与预期不符。阅读源码发现，经过层层调用后在最终返回结果的地方对应用的权限进行了校验，对于没有权限的应用返回值是没有意义的缺省值，否则返回实际值起到保护用户的隐私目的。

# 2. 反射的原理

我们在平时写代码的时候，创建的类(`class`)文件都以`.java`为后缀名存储的，之后它会被编译为以`.class`为后缀的文件，最后被执行的其实是`.class`文件。

除了基本数据类型以外，Java的其它类型全部都是`class`(包括`interface`)，这也是为什么说在Java中，万物皆对象，例如`String`，`Object`，`Runnable`等，我们可以说，`class`(包括`interface`)的本质就是数据类型。

而`class`是由JVM在执行过程中动态加载的。JVM在第一次读取到一种`class`类型时，将其加载进内存。加载一种`class`，JVM就为其创建一个`Class`类型的实例，并关联起来。

注意：这里的`Class`类型是一个名叫`Class`的`class`。它长这样：

```
public final class Class {
    private Class() {}
}
```

以`String`类为例，当JVM加载`String`类时，它首先通过类加载器加载`String.class`文件到内存，然后，为`String`类创建一个`Class`实例并关联起来：

```
Class clazz = new Class(String);
```

这个`Class`实例是JVM内部创建的，如果我们查看JDK源码，可以发现`Class`类的构造方法是`private`，只有JVM能创建`Class`实例，Java程序是无法创建`Class`实例的。

由于JVM为每个加载的`class`创建了对应的`Class`实例，并在实例中保存了该`class`的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个`Class`实例，我们就可以通过这个`Class`实例获取到该实例对应的`class`的所有信息。

这种通过`Class`实例获取`class`信息的方法称为反射。

# 3. 反射的使用

## 3.1 通过反射获取属性字段

### 3.1.1 获取Class对象

之前说我们可以通过Class对象获取这个类的信息，那么获得这个类的字段肯定也要通过Class对象，我们一共有三种方式获取Class对象：

1. 通过Object类中的getClass()方法

```java
Person p = new Person();
Class c = p.getClass();
```

2. 通过.class 获取到字节码文件对象

```java
Class c2 = Person.class;
```

3. 通过Class类中的方法

```java
Class c3 = Class.forName("Person");
```

前两种必须明确Person类型，后面是指定这种类型的字符串就行，这种扩展更强，我不需要知道你的类，我只提供字符串，按照配置文件加载就可以了。

### 3.1.2 获取属性字段

```java
public class ReflectionDemo {

	public static void main(String[] args) throws NoSuchFieldException, SecurityException {
		Class collegeStudentClass = CollegeStudent.class;
		//获取继承的public属性
		System.out.println(collegeStudentClass.getField("gender"));
		//获取private属性
		System.out.println(collegeStudentClass.getDeclaredField("age"));
		//获取类中定义的public属性
		System.out.println(collegeStudentClass.getField("studentId"));
		//获取所有属性
		Field[] fields = collegeStudentClass.getDeclaredFields();
		for(Field field:fields) {
			System.out.println(field);
		}
	}

}
/**
 * 结果:public java.lang.String dev.wenbo.reflection.Person.gender
private int dev.wenbo.reflection.CollegeStudent.age
public java.lang.String dev.wenbo.reflection.CollegeStudent.studentId

public java.lang.String dev.wenbo.reflection.CollegeStudent.studentId
private int dev.wenbo.reflection.CollegeStudent.age
 */
```

我们看到，获取`public`属性和`private`属性的方法不一样，使用`getField()`获取`public`属性，而通过`getDeclaredField()`获取`private`属性。实际上，使用`getDeclaredField()`可以获取任意属性，而`getField()`只能获取`public `属性。

我们通过`getDeclaredFields()`返回了一个包含类中定义的属性的数组，实际结果是并不能获得从父类继承下来的属性，只能获得这个类自己定义的属性。

一个`Field`对象包含了一个字段的所有信息：

- `getName()`：返回字段名称，例如，`"name"`；
- `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
- `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

### 3.1.2 获取属性字段值

我们可以更进一步，通过`Field`变量来获得属性值：

```java
public class ReflectionDemo {

	public static void main(String[] args) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, InstantiationException {
		Class collegeStudentClass = CollegeStudent.class;
		//获取继承的public属性
		System.out.println(collegeStudentClass.getField("gender"));
		//获取private属性
		System.out.println(collegeStudentClass.getDeclaredField("age"));
		//获取类中定义的public属性
		System.out.println(collegeStudentClass.getField("studentId"));
		//获取所有属性
		Field[] fields = collegeStudentClass.getDeclaredFields();
		for(Field field:fields) {
			System.out.println(field);
		}
		//获取属性值
		CollegeStudent c = (CollegeStudent)collegeStudentClass.newInstance();
		Field ageField = collegeStudentClass.getDeclaredField("age");
		ageField.setAccessible(true);
		int age1 = (int) ageField.get(c);
		System.out.println(age1);
	}

}
/**
 * 结果:public java.lang.String dev.wenbo.reflection.Person.gender
private int dev.wenbo.reflection.CollegeStudent.age
public java.lang.String dev.wenbo.reflection.CollegeStudent.studentId
public java.lang.String dev.wenbo.reflection.CollegeStudent.studentId
private int dev.wenbo.reflection.CollegeStudent.age
23
 */
```

因为我们把`age`定义为`private`变量，所以我们通过`Field`获得这个变量之后要使用`setAccessible(true)`方法才能得到这个变量定义的值，也就是23。