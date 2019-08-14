---
title: builder-pattern
date: 2019-02-13 15:22:25
tags: Design Pattern
index_img: ../post_img/taoyifenxi.jpeg
---

# 建造者模式

## 定义

将一个复杂对象的构造与它的表示分离，使同样的构建过程可以创建不同的表示，这样的设计模式被称为建造者模式。

## 特点

### 优点

1. 各个具体的建造者相互独立，有利于系统的扩展。
2. 客户端不必知道产品内部的组成细节，便于控制细节风险。

### 缺点

1. 产品的组成部分必须相同，这限制了其使用范围。
2. 如果产品的内部变化复杂，该模式会增加很多的建造者类。

建造者模式和工厂模式的关注的不同，建造者模式注重零部件的组装过程，而工厂方法模式更注重零部件的创建过程，但两者可以结合使用。


## 结构

### 产品角色

他是包含多个组成部分的复杂对象，是由建造者来创建其各个组成部件。

### 抽象建造者

它是一个包含创建产品各个子部件的抽象方法的 接口，通常还包含一个返回复杂产品的方法。

### 具体建造者

实现 Builder 接口，完成复杂产品的各个部件的具体创建方法。

### 指挥者

它调用建造者对象中的部件构造与装配方法完成复杂对象的创建，在指挥者中不涉及具体产品的信息。

## 应用场景

1. 创建的对象较复杂，由多个部件构成，各部件面临着复杂的变化，但构件间的建造顺序是稳定的。
2. 创建复杂对象的算法独立于该对象的组成部分已经它们的装配方式，即产品的构建过程和最终的表示是独立的。

### 扩展

建造者模式在应用过程中可以根据需要改变，如果创建的产品种类只有一种，只需要一个具体的建造者，这是可以省略掉抽象建造者，甚至可以忽略指挥者的角色。

## 代码实现
