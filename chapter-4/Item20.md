# 优先选择接口而不是抽象类

Java有两种机制去定义一个允许多个实现的类型：接口和抽象类。由于Java 8 [JLS 9.4.3]中为接口引入了默认方法，所以这两种机制都允许你为一些实例方法提供实现。一个主要区别是，要实现抽象类定义的类型，该类必须是抽象类的子类。因为Java只允许单继承，所以这种对抽象类的限制严重限制了它们作为类型定义的使用。任何定义了所有必需的方法并遵守通用约定的类都允许实现接口，而不管类属于类层次结构中的何处。

**可以很容易地对现有类进行改造，以实现新的接口**。你所要做的就是添加所需的方法(如果它们还不存在的话)，并在类声明中添加一个`implementation`子句。比如，许多已经存在的类当初在添加到平台时，被改进成实现了`Comparable`，`Iterable`和`Autocloseable`接口。一般来说，现有的类不能被修改成去继承一个新的抽象类。如果你想让两个类继承同一个抽象类，你必须把它放在类型层次结构的高层，在那里它是两个类的祖先。不幸的是，这可能会对类型层次结构造成巨大的附带损害，迫使新抽象类的所有后代对其进行子类化，无论它是否合适。

接口是定义`mixins`的完美工具。总的来说，`mixins`是一种类型，即一个类除了实现它的主类型外，还可以声明它提供一些可选的行为。例如，`Comparable`是一个`mixin`接口，它允许一个类声明它的实例可以跟其他可相互比较的对象进行排序。这样的接口称为`mixin`，因为它允许将可选功能“混入”到类型的主要功能里。抽象类不能用于定义`mixins`，原因与它们无法改装成现有类一样：一个类不能有多个父类，并且类层次结构中没有合理的位置来插入`mixin`。

接口允许构造非层次化类型框架。类型层次结构对于组织一些事情很好，但是其他事情不能整齐地归入严格的层次结构。比如，假设我们有一个代表歌手的接口和另一个代表词曲作者的接口：

```java
public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}
```

在现实生活中，一些歌手也是词曲作者。因为我们使用接口而不是抽象类来定义这些类型，所以完全允许单个类同时实现`Singer`和`Songwriter`。事实上，我们可以定义第三个接口，同时继承`Singer`和`Songwriter`接口，并添加一个新的恰当的组合方法：

```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```

你并不总是需要这种级别的灵活性，但是当你需要时，接口就是救命稻草。另一个选择是一个臃肿的类层次结构，它给每一个支持的属性组合提供一个单独的类。如果类型系统中有n个属性，那么可能需要支持的组合有2^n个。这个被称作『组合激增』。臃肿的类层次结构可能导致臃肿的类，这些类中的很多方法仅仅是它们的参数类型上不同，因为在类层次结构中没有用于捕获公共行为的类型。

**接口通过包装器类做法(项目18)能够支持安全、强大的功能增强**。如果你使用抽象类来定义类型，那么你将让希望添加功能的程序员除了继承之外别无选择。结果生成的类不如包装类强大，也更脆弱。

当一个接口方法相对于其他接口方法而言有明显的实现时，请考虑以默认方法的形式提供一个实现，以此来帮助程序员。有关此技术的示例，请参阅第104页的`removeIf`方法。如果你提供了默认方法，确保在继承的时候使用JavaDoc标签来记录他们。	尽管许多接口都指定了`Object`方法（如`equals`和`hashCode`）的行为，但是不允许为它们提供默认方法。而且，不允许接口包含实例字段或非公共静态成员（私有静态方法除外）。最后，你不能往不受你控制的接口里加默认方法。