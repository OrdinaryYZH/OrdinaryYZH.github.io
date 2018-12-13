# 第8章 生成器模式(Builder)

> 生成器模式的定义：
>
> 将一个复杂对象的构建与它的表示分离，是的同样的构建过程可以创建不同的表示
>
> ![](https://ws1.sinaimg.cn/large/8747d788gy1fy55skxzoxj20wv0jz0to.jpg)

## 8.3 模式讲解

### 8.3.1 认识生成器模式

![](https://ws1.sinaimg.cn/large/8747d788gy1fy4e510k68j21kw0qkakc.jpg)

#### 1. 生成器模式的功能

> 构建复杂的产品！！
> 而且更重要的是，这个构建过程是不变的，变化的部分放到了具体的Builder部分了，只要配置不同的Builder，那么同样的构建过程，就能构建出不同的产品。

#### 2. 生成器模式的构成

不管如何变化，Builder模式都存在这么两个部分：

1. 部件构造和产品装配(Builder接口及其实现)
2. 整体构建的算法(Director)；在director实现算法时，遇到需要创建和组合具体部件的时候，会委托Builder来完成

#### 3. 生成器模式的使用

```java
Builder builder = new ConcreteBuilder();
Director director = new Director(builder);
director.construct(); // 貌似传统的construct方法是没有返回product的
Product product = builder.build();
```

#### 4. 生成器模式的调用顺序示意图

![](https://ws1.sinaimg.cn/large/8747d788gy1fy4eywp03jj216e0rxtfi.jpg)

### 8.3.2 生成器模式的实现（略）

有点"难懂"，有点啰嗦；略过

### 8.3.3 使用生成器模式构建复杂对象（略）

### 8.3.4 生成器模式的优点

1. 松散耦合（因为把产品的构建算法跟具体表示分离开，构建算法可复用，具体的产品表现可灵活切换）
2. 可以很容易地改变产品的内部表示
3. 更好的复用性

### 8.3.5 思考生成器模式

#### 1. 生成器模式的本质

分离整体构建算法和部件构造

#### 2. 何时选用生成器模式

* 如果创建对象的算法，应该独立于该对象的组成部分以及他们的装配方式时
* 如果同一个构建过程有着不同的表示时

### 8.3.6 相关模式



### 补充：简化版Builder模式

1. 客户端与director合并，让客户端自己设计整体构建
2. 去掉Builder接口
3. Product和Builder合并，即使用静态内部类表示Builder