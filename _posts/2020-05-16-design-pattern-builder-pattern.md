---
layout: post
title: 设计模式之建造者模式(Builder Pattern)
categories: DesignPattern
description: 建造者模式介绍
keywords: DesignPattern, 设计模式
---

建造者模式(Builder Pattern)是将一个复杂的对象的**构建**与它的**表示**分离，使得同样的构建过程可以创建不同的表示。

创建者模式隐藏了复杂对象的创建过程，它把复杂对象的创建过程加以抽象，通过子类继承或者重载的方式，动态的创建具有复合属性的对象。它是一种**创建型模式**。

# 1. 建造者模式的实现

## 1.1 建造者模式实现图

![建造者模式实现图](/images/posts/designpattern/Builder_UML_class_diagram.png)

## 1.2 具体实现

如上图所示，builder模式有4个角色。

- Product: 最终要生成的对象，例如 Computer实例。
- Builder： 构建者的抽象基类（有时会使用接口代替）。其定义了构建Product的抽象步骤，其实体类需要实现这些步骤。其会包含一个用来返回最终产品的方法`Product getProduct()`。
- ConcreteBuilder: Builder的实现类。
- Director: 决定如何构建最终产品的算法. 其会包含一个负责组装的方法`void Construct(Builder builder)`， 在这个方法中通过调用builder的方法，就可以设置builder，等设置完成后，就可以通过builder的 `getProduct()` 方法获得最终的产品。

### 1.2.1 创建目标Computer类

```java
public class Computer {
	private String cpu;//必须
    private String ram;//必须
    private int usbCount;//可选
    private String keyboard;//可选
    private String display;//可选

    public Computer(String cpu, String ram) {
        this.cpu = cpu;
        this.ram = ram;
    }
    public void setUsbCount(int usbCount) {
        this.usbCount = usbCount;
    }
    public void setKeyboard(String keyboard) {
        this.keyboard = keyboard;
    }
    public void setDisplay(String display) {
        this.display = display;
    }
    @Override
    public String toString() {
        return "Computer{" +
                "cpu='" + cpu + '\'' +
                ", ram='" + ram + '\'' +
                ", usbCount=" + usbCount +
                ", keyboard='" + keyboard + '\'' +
                ", display='" + display + '\'' +
                '}';
    }
}
```

## 1.2.2 创建ComputerBuilder抽象类

```
public abstract class ComputerBuilder {
	public abstract void setUsbCount();
    public abstract void setKeyboard();
    public abstract void setDisplay();

    public abstract Computer getComputer();
}
```

### 1.2.3 创建实体创建者类

```java
public class MacComputerBuilder extends ComputerBuilder{
	private Computer computer;
    public MacComputerBuilder(String cpu, String ram) {
        computer = new Computer(cpu, ram);
    }
    @Override
    public void setUsbCount() {
        computer.setUsbCount(2);
    }
    @Override
    public void setKeyboard() {
        computer.setKeyboard("苹果键盘");
    }
    @Override
    public void setDisplay() {
        computer.setDisplay("苹果显示器");
    }
    @Override
    public Computer getComputer() {
        return computer;
    }
}

public class LenovoComputerBuilder extends ComputerBuilder{
	private Computer computer;
    public LenovoComputerBuilder(String cpu, String ram) {
        computer=new Computer(cpu,ram);
    }
    @Override
    public void setUsbCount() {
        computer.setUsbCount(4);
    }
    @Override
    public void setKeyboard() {
        computer.setKeyboard("联想键盘");
    }
    @Override
    public void setDisplay() {
        computer.setDisplay("联想显示器");
    }
    @Override
    public Computer getComputer() {
        return computer;
    }
}
```

### 1.2.4 创建指导者类

```java
public class ComputerDirector {
	public void makeComputer(ComputerBuilder builder){
        builder.setUsbCount();
        builder.setDisplay();
        builder.setKeyboard();
    }
}
```

### 1.2.5 测试

```java
public class BuilderDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ComputerDirector director=new ComputerDirector();//1
        ComputerBuilder builder=new MacComputerBuilder("I5处理器","三星125");//2
        director.makeComputer(builder);//3
        Computer macComputer=builder.getComputer();//4
        System.out.println("mac computer:"+macComputer.toString());

        ComputerBuilder lenovoBuilder=new LenovoComputerBuilder("I7处理器","海力士222");
        director.makeComputer(lenovoBuilder);
        Computer lenovoComputer=lenovoBuilder.getComputer();
        System.out.println("lenovo computer:"+lenovoComputer.toString());
	}

}
```

可以看到，director的实际作用是利用builder组装computer，而最后要获得的对象是通过builder获得的。

# 2. 创建者模式分析

## 2.1 优点

- 使用建造者模式可以使客户端不必知道产品内部组成的细节。
- 具体的建造者类之间是相互独立的，这有利于系统的扩展。
- 具体的建造者相互独立，因此可以对建造的过程逐步细化，而不会对其他模块产生任何影响。

## 2.2 缺点

- 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似；如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。

- 如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大。

## 2.3 适用场景

- 隔离复杂对象的创建和使用，相同的方法，不同执行顺序，产生不同事件结果

- 多个部件都可以装配到一个对象中，但产生的运行结果不相同

- 产品类非常复杂或者产品类因为调用顺序不同而产生不同作用

- 初始化一个对象时，参数过多，或者很多参数具有默认值

- Builder模式不适合创建差异性很大的产品类，产品内部变化复杂，会导致需要定义很多具体建造者类实现变化，增加项目中类的数量，增加系统的理解难度和运行成本

- 需要生成的产品对象有复杂的内部结构，这些产品对象具备共性