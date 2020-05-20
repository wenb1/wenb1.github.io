---
layout: post
title: Java之注解(Annotations)
categories: Java
description: 讲解Java中的注解
keywords: Java, annotations, 注解
---

注解用于为 Java 代码提供元数据。作为元数据，注解不直接影响你的代码执行。注解在Java SE5中加入到Java中，提供对代码解释和说明的功能。

只看官方定义很懵，不知所云。其实注解就像生活中商品的标签，在超市里，我们能看到各种各样的糕点，有蛋糕，面包，曲奇等等，这些标签名字其实就像注解一样，解释说明了这些糕点的类型，我们也能通过这些标签大概知道这些糕点的口感，比如面包就是软软的，曲奇是脆脆的。我们还能贴上更具体的标签，像草莓蛋糕，巧克力曲奇，这样解释说明的效果会更好。

# 1. 注解的使用格式

我们只需要在要注解的类或者方法上使用`@+注解名`就能完成注解，如：

```java
@Test
public class Test {
}

@Test
public void Test(){
    
}
```

加了注解的方法和没加注解的方法本质上没有什么不同。

# 2. Java内置注解

Java内置了三个注解可供使用

| 内置注解名        | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| @Override         | 限定重写父类方法，若想要重写父类的一个方法时，需要使用该注解告知编译器我们正在重写一个方法。如此一来，当父类的方法被删除或修改了，编译器会提示错误信息；或者该方法不是重写也会提示错误。 |
| @Deprecated       | 标记已过时，当我们想要让编译器知道一个方法已经被弃用时，应该使用这个注解。Java推荐在javadoc中提供信息，告知用户为什么这个方法被弃用了，以及替代方法是什么。 |
| @SuppressWarnings | 抑制编译器警告，该注解仅仅告知编译器，忽略它们产生了特殊警告。 |

# 3. 定义一个注解

我们可以通过`@interface`关键字来创建一个自己的注解，注意：注解是不支持继承的，因此不能使用关键字extends来继承某个@interface

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test(){}
```

注意到，我们定义一个注解的时候，和定义一个接口非常相似。只不过定义注解时，我们不能只使用`@interface`关键字来实现注解定义，还需要使用`@Target`和`@Retention`这些**元注解(meta-annotations)**来完成自定义注解创建。

## 3.1 元注解

下面讲讲什么是元注解(meta-annotations)：

元注解一共有五个，`@Retention`、 `@Target`、 `@Document`、 `@Inherited`和`@Repeatable`，元注解的作用是注解你要创建的注解，就像原始天尊，创造万事万物，元注解创造其它注解。

1. `@Retention`

   它表示注解存在阶段是保留在源码（编译期），字节码（类加载）或者运行期（JVM中运行）。在@Retention注解中使用枚举RetentionPolicy来表示注解保留时期。

   - @Retention(RetentionPolicy.SOURCE)，注解仅存在于源码中，在class字节码文件中不包含

   - @Retention(RetentionPolicy.CLASS)， 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得

   - @Retention(RetentionPolicy.RUNTIME)， 注解会在class字节

     码文件中存在，在运行时可以通过反射获取到

2. `@Target`

   @Target元注解表示我们的注解作用的范围就比较具体了，可以是类，方法，方法参数变量等，同样也是通过枚举类ElementType表达作用类型。

   - @Target(ElementType.TYPE) 作用接口、类、枚举、注解

   - @Target(ElementType.FIELD) 作用属性字段、枚举的常量

   - @Target(ElementType.METHOD) 作用方法

   - @Target(ElementType.PARAMETER) 作用方法参数

   - @Target(ElementType.CONSTRUCTOR) 作用构造函数

   - @Target(ElementType.LOCAL_VARIABLE)作用局部变量

   - @Target(ElementType.ANNOTATION_TYPE)作用于注解（@Retention注解中就使用该属性）

   - @Target(ElementType.PACKAGE) 作用于包

   - @Target(ElementType.TYPE_PARAMETER) 作用于类型泛型，即泛型方法、泛型类、泛型接口 （jdk1.8加入）

   - @Target(ElementType.TYPE_USE) 类型使用.可以用于标注任意类型除了 class （jdk1.8加入）

3. `@Documented`

   它的作用是能够将注解中的元素包含到 Javadoc 中去.

4. `@Inherited`

   一个被`@Inherited`注解了的注解修饰的一个父类，如果他的子类没有被其他注解修饰，则它的子类也继承了父类的注解。

5. `@Repeatable` (Java 1.8引入)

   被这个元注解修饰的注解可以同时作用一个对象多次，但是每次作用注解又可以代表不同的含义。

   例子：

   ```java
   @interface Persons {
       Person[]  value();
   }
   
   @Repeatable(Persons.class)
   @interface Person{
       String role default "";
   }
   
   @Person(role="artist")
   @Person(role="coder")
   @Person(role="student")
   public class Man{
   }
   ```

我们看到`Test`注解中没有定义方法或者变量，这种注解叫做标记注解(marker annotation)，而这个`Repeatable`的例子中我们定义了一些属性。实际上，我们可以在自定义的注解里定义属性，让注解的作用更大。

## 3.2 注解的属性

注解的属性其实和类中定义的变量有异曲同工之处，只是注解中的变量都是成员变量（属性），并且注解中是没有方法的，只有成员变量，变量名就是使用注解括号中对应的参数名，变量返回值注解括号中对应参数类型。我们可以给属性规定一个默认值。

编译器对元素的默认值有些挑剔。首先，元素不能有不确定的值。也就是说，元素必须要么具有默认值，要么在使用注解时提供元素的值。其次，对于非基本类型的元素，无论是在源代码中声明，还是在注解接口中定义默认值，都不能以null作为值，这就是限制，但造成一个元素的存在或缺失状态，因为每个注解的声明中，所有的元素都存在，并且都具有相应的值，为了绕开这个限制，只能定义一些特殊的值，例如空字符串或负数，表示某个元素不存在。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
	public String description() default "Hello";
}
```

### 3.2.1 注解的属性类型

- 所有基本数据类型
- String
- enum
- Class
- 注解
- 以上类型的一维数组类型

### 3.2.2 注解成员变量赋值

我们可以在使用注解时给注解的成员变量赋值：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
	public int id();
	public String description() default "Hello";
}

public class TestDemo {
	
	@Test(id=1, description="This is test")
	public void helloTest() {
	}
}
```

# 4. 注解的本质

注解的本质就是一个Annotation接口：

```java
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    Class<? extends Annotation> annotationType();
}
```

注解本身就是Annotation接口的子接口，也就是说注解中其实是可以有属性和方法，但是接口中的属性都是static final的，对于注解来说没什么意义，而我们定义接口的方法就相当于注解的属性，也就对应了前面说的为什么注解只有属性成员变量，其实他就是接口的方法，这就是为什么成员变量会有括号，不同于接口我们可以在注解的括号中给成员变量赋值。

# 5. 注解的作用

- 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息
- 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。
- 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取
  值得注意的是，注解不是代码本身的一部分。

**参考文章**

[Java 注解完全解析 (若丨寒)](https://www.jianshu.com/p/9471d6bcf4cf)

[java注解-最通俗易懂的讲解 (Tanyboye)](https://blog.csdn.net/qq1404510094/article/details/80577555)

[深入理解Java注解类型(@Annotation) (zajian_)](https://blog.csdn.net/javazejian/article/details/71860633)

[Java注解-元数据、注解分类、内置注解和自定义注解 (乐字节)](https://segmentfault.com/a/1190000019887623)