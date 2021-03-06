# 面向对象的23种设计模式（JAVA）

## 一、什么是设计模式

关于设计模式的定义，可以从以下两个方面来说明。

### 1. 软件设计模式的概念

软件设计模式（Software Design Pattern），又称设计模式，是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。它描述了在软件设计过程中的一些不断重复发生的问题，以及该问题的解决方案。也就是说，它是解决特定问题的一系列套路，是前辈们的代码设计经验的总结，具有一定的普遍性，可以反复使用。其目的是为了提高代码的可重用性、代码的可读性和代码的可靠性。

### 2. 学习设计模式的意义

*设计模式的本质是面向对象设计原则的实际运用，是对类的封装性、继承性和多态性以及类的关联关系和组合关系的充分理解。*正确使用设计模式具有以下优点。

- 可以提高程序员的思维能力、编程能力和设计能力；
- 使程序设计更加标准化、代码编制更加工程化，使软件开发效率大大提高，从而缩短软件的开发周期。
- 使设计的代码可重用性高、可读性强、可靠性高、灵活性好、可维护性强。

当然，软件设计模式只是一人引导。在具体的软件开发中，必须根据设计的应用系统的特点和要求来恰当选择。对于简单的程序开发，可能写一个简单的算法要比引入某种设计模式更加容易。但对大项目的干他看者框架设计，用设计模式来组织代码显然更好。

### 3. 软件设计模式的基本要素

软件设计模式使人们可以更加简单方便地复用成功的设计和体系结构，它通常包含以下几个基本要素：



## 二、软件开发设计原则

### 1. 开闭原则

*开闭原则（Open Closed Principle, OCP）*由勃兰特·梅耶提出，他在1988年的著作《面向对象软件构造》（Object Oriented Software Construction）中提出：软件实体应当对扩展开放，对修改关闭（Software entities should be open for extension, but closed for modification），这就是开闭原则的经典定义。

### 2. 里氏替换原则

*里氏替换原则（Liskov Substitution Principle，LSP）*由麻省理工学院计算机科学实验室的里斯科夫（Liskov）女士在1987年的“面向对象技术的高峰会议”（OOPSLA）上发表的一篇文章《数据抽象和层次》（Data Abstraction and Hierarchy）里提出来的，他提出：继承必须确保超类所拥有的性质在子类中 仍然成立（Inheritance should ensure that any property proved about supertype objects also holds for subtype object）。

#### 里氏替换原则的作用

里氏替换原则的主要作用如下。

1. 里氏替换原则是实现开闭原则的重要方式之一；
2. 它克服了继承中重写父类造成的可复用性变差的缺点；
3. 它是动作正确性的保证。即类的扩展不会经已有的系统引入新的错误，降低了代码出错的可能性；
4. 加强程序的健壮性，同时变更时可以做到非常好的兼容性，提高程序的维护性、可扩展性，降低需求变更时引入的风险。

## 二、行为型模式

### 1. 访问者模式（Visitor）

### 2. 模板方法模式（Template Method）

### 3. 策略模式（Strategy）

### 4. 状态模式（State）

### 5. 观察者模式（Observer）

### 6. 备忘录模式（Memento）

### 7. 中介者模式（Mediator）

### 8. 迭代器模式（Iterator）

### 9. 解释器模式（Interpreter）

### 10. 命令模式（Command）

### 11. 责任链模式（Chain of Responsibility）



## 三、创建型模式

### 1. 单例模式（Singleton）

### 2. 工厂方法模式（Factory Method）

### 3. 抽象工厂方法模式（Abstract Factory）

### 4. 建造者模式（Builder）

### 5. 原型模式（Prototype）



## 四、结构型模式

### 1. 适配器模式（Adapter）

### 2. 桥接模式（Bridge）

### 3. 组合模式（Composite）

### 4. 装饰模式（Decorator）

### 5. 门面模式（Facade）

### 6. 享元模式（Flyweight）

比较典型的就是连接池，先创建出若干对象，应用在需要使用时，到容器中去取，使用后还给容器。可以理解为一些元数据，供大家共享。

### 7. 代理模式（Proxy）





