# 模版方法

## 定义

定义一个操作中的算法骨架，而将算法的一些步骤延迟到子类中，使得子类可以不改变该算法的情况下冲定义该算法的某些特定步骤，它是一种类行为模式。

## 特点

### 优点

1. 他封装了不变部分，扩展可变部分。它把认为是不变部分的算法封装到了父类中实现，而把可变部分算法由子类继承实现，便于子类继续扩展。
2. 它在父类中提取了公共的部分代码，便于代码复用。
3. 部分方法是由子类实现的，因此子类可以通过扩展方式增加相应的功能，符合开闭原则。

### 缺点

1. 对每个不同的实现都需要定义一个子类，这会导致类的个数增加，系统更加庞大，设计也更加抽象。
2. 父类中的抽象方法由子类实现，子类的执行结果会影响父类的结果，这导致一种反向的控制结构，提高了代码的阅读难度。

## 结构

### 抽象类

负责给出一个算法的骨架，它由一个模版方法和若干个基本方法构成。

#### 模版方法

定义了算法的骨架，按照某种顺序调用其包含的基本方法。

#### 基本方法

是整个算法中的一个步骤，

1. 抽象方法：在抽象类中声明，由具体子类实现
2. 具体方法：在抽象类中已经实现，在具体子类可以继承或者重写它
3. 钩子方法：在抽象类中已经实现，包括用于判断的逻辑方法，和需要子类重写的空方法。

### 具体子类

实现抽象类中所定义的抽象方法和钩子方法。

## 应用场景

1. 算法的整体步骤很固定，但其中个别部分易变时，这时候可以使用模版方法，将容易变的部分抽象出来，供子类实现。
2. 当多个子类存在公共的行为时，可以将其提取出来并集中到一个公共父类中以避免代码重复。
3. 当需要控制子类的扩展时，模版方法只在特定点调用钩子操作。

### 扩展

可以使用 钩子方法 使得子类控制父类的行为。

## 代码实现
