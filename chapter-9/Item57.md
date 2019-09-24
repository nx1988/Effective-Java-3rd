# 最小化局部变量的作用域

从本质上来说，本条款类似于条款15：最小化类与成员的可访问性。通过最小化局部变量的作用域，我们可以增加代码的可读性与可维护性，同时还会降低出错的可能性。

诸如C语言之类的老式编程语言会强制要求将局部变量声明在块的头部，一些程序员就继续遵循着这个习惯。不过，这是个值得打破的习惯。友情提示一下，Java可以在语句合法的任何地方声明变量（从C99开始，C语言也可以这样做了）。

用于最小化局部变量作用域的最为强大的技术就是在首次需要时再进行声明。如果变量在使用前声明，那就会有些杂乱——对于那些想要搞清楚程序用途的读者来说，这会分散他们的注意力。等到使用变量时，读者可能就记不得变量的类型或是初始值了。

过早声明局部变量不仅会导致其作用域过早开始，还会造成过晚结束的后果。局部变量的作用域从其声明之处开始，一直延续到外层块结束为止。如果变量声明在其所需要使用的块的外部，那么当程序退出这个块时，它还依旧可见。如果不小心在应该使用变量的区域的前面或者后面用到了这个变量，那后果就是灾难性的了。

**几乎每个局部变量声明都应该包含一个初始化器**。如果没有足够的信息来初始化变量，那就应该将声明推迟到信息足够之时。该原则的一个例外情况是`try-catch`语句。如果变量被初始化为一个表达式，而该表达式的计算可能会抛出一个检查异常，那就应该在`try`块中初始化该变量（除非外层方法可以传播该异常）。如果值是在`try`块外使用的，那么它就需要在`try`块之前进行声明，而在这里进行声明就是所谓的『不太聪明的初始化』。比如说，请参阅第283页。

循环展现出了最小化变量作用域的一种特殊机会。`for`循环（无论是传统的还是`for-each`形式）可以声明循环变量，并将其作用域限制为所需的精确区域（该区域包含了了循环体以及`for`关键字与体之间的圆括号中的代码）。因此，如果循环变量的内容在循环终止后就用不上了，那么相比于`while`循环来说，请优先选择`for`循环。

比如说，下面是迭代集合的一种推荐做法（条款58）：

```java
// Preferred idiom for iterating over a collection or array
for (Element e : c) {
... // Do Something with e
}
```

如果你需要访问迭代器，可能是想调用它的`remove`方法，那么首选的习惯用法是使用传统的`for`循环来代替`for-each`循环：

```java
// Idiom for iterating when you need the iterator
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
	Element e = i.next();
	... // Do something with e and i
}
```

为了搞明白为什么`for`循环比`while`循环好，请考虑下面的代码片段，其中包含两个`while`循环和一个`bug`：

```java
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
	doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG!
	doSomethingElse(i2.next());
}
```

第二个循环出现了复制粘贴错误：它初始化了一个新的循环变量`i2`，但是使用的是旧的变量`i`，不幸的是，它还在作用域中。最后，代码编译没有出错，并且运行的时候也没有抛出异常，但是的确做了的一件错事。第二个循环会立马终止，而不会遍历`c2`，从而产生`c2`为空的错误印象。因为程序会无声地出错，所以很长一段时间内都无法检测到该错误。

如果相似的复制粘贴错误与`for`循环（`for-each`或者传统的）一起出现，结果是代码不会编译通过。第一个循环中的元素（或迭代器）变量不在第二个循环中的作用域中。下面是它在传统`for`循环里使用的例子：

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
	Element e = i.next();
	... // Do something with e and i
}
...
    
// Compile-time error - cannot find symbol i
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
	Element e2 = i2.next();
	... // Do something with e2 and i2
}
```

而且，你使用`for`循环，能减少复制粘贴带来的错误的可能性。因为两个循环中没有动机使用不同的变量名。

循环是完全独立的，所以重用元素(或迭代器)变量名没有害处。事实上，这样做通常很流行。`for`循环比`while`循环还有一个优点：它更短，这增强了可读性。

下面是另一个循环的习惯用法，它最小化了局部变量的范围：

```java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
	... // Do something with i;
}
```

关于这个用法需要注意的重要一点是，它有两个循环变量，`i`和`n`，它们都具有完全正确的作用域。第二个变量`n`用于存储第一个变量的限制，从而避免了每次迭代中冗余计算的代价。作为一个规则，如果循环测试涉及一个方法调用，并且保证在每次迭代中返回相同的结果，那么应该使用这个习惯用法。

最小化局部变量范围的最后一种技术是保持方法小而集中。如果你在同一方法中组合了两个活动，那么其中一个活动相关的局部变量可能位于执行另一个活动的代码的范围内。为了防止这种情况发生，你可以将这个方法一分为二：一个活动一个方法。