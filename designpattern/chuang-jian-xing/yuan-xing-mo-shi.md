# 原型模式

## 定义

用一个已经创建的实例作为原型，通过复制改原型对象来创建一个和原型相同或相似的新对象。在这里，原型实例指定了要创建的对象的种类。用这种方式创建对象非常高效，根本无须知道对象创建的细节。

## 结构

### 抽象原型 

规定了具体原型对象必须实现的接口。

### 具体原型类

实现抽象原型类的 clone 方法，它是可被复制的对象。

### 访问类

使用具体原型类中的 clone 方法来复制新的对象。

## 特点

### 优点

### 缺点

### 模式的实现

1.  原型模式分为深克隆和浅克隆，Java 中的 clone 是浅克隆。

## 应用场景

1. 对象之间相同或者相似，即只是个别的几个属性不同的时候。
2. 对象的创建过程比较麻烦，但复制比较简单的时候。

### 扩展

1. 原型模式可扩展为带原型管理器的原型模式，它在原型模式的基础上增加了一个原型管理器类。该类用 HashMap 保存多个复制的原型。Client 类可以通过管理器的 get 方法从中获取复制的原型。

## 代码实现


