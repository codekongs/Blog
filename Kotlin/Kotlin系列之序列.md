> 今天来一起学习下Kotlin中的序列。

在开始之前，先说一下我们定义的演示数据，后面数据定义的代码就不重复出现了。

```kotlin
//Person数据类
data class Person(val name: String, val age: Int)	

val personList = listOf(Person("小红", 22),
        Person("小明", 23),
        Person("小白", 22))			
```

上面定义了一个Person的数据类和一个List，后面的操作都是对这个List进行操作。

#### 什么是序列

如果你了解过Java8或者Python，那可以把这里的序列简单理解为Java8中的流或者是Python中的生成器。如果不了解也没事，序列简单一点说就是实现对集合的操作进行延迟，专业的说法叫惰性集合操作，就是把你对集合的一系列操作拖到最后一刻才执行。这么看序列就是个拖延症晚期。

我们通过代码来看一下我们为什么需要序列，我们有一个需求是过滤出一个Person的集合中的所有名字长度为2的名字，我们可以使用下面的代码。

```kotlin
fun main(args: Array<String>) {
    println(personList.map { it.name }.filter { it.length == 2 })
}
```

输出结果如下：

```
[小红, 小明, 小白]
```

上面的代码有什么缺陷呢？根据Kotlin的定义，上面的`map`操作会生成一个所有人名字的集合，然后`filter`操作又会对这个生成的中间集合做一次操作，生成最终的符合条件的集合。这时候你就会发现整个运算过程生成了两个集合，但是其中的中间产生的集合其实是我们不需要的，我们只关注最终的结果，中间生成的集合会占用内存，降低我们程序的性能，序列的出现就是为了解决这样一类问题。

#### 创建序列

下面来看看我们如何创建一个序列。序列的创建方式分为两种，一种是将一个现存的集合变为一个序列，一种是直接创建一个序列。下面就来一起看看这两种操作。

##### 将集合变为序列

我们对一个集合调用`asSequence`函数就会将一个集合变为一个序列，就像下面这样：

```kotlin
fun main(args: Array<String>) {
    println(personList.asSequence())
}
```

这时候如果你使用`println`输出这个序列，它的结果如下：

```
kotlin.collections.CollectionsKt___CollectionsKt$asSequence$$inlined$Sequence$1@63961c42
```

变成序列之后我们对其进行集合相关的操作就不会生成中间集合了。

##### 直接创建一个序列

我们使用`generateSequence`函数就可以创建一个序列。比如自然数序列。我们用一个List是无法表示所有自然数的，但是我们却可以使用序列去表示它。创建一个序列的要求是给定序列中的前一个元素，并且给定一个可以计算出下一个值的函数即可，就像下面这个表示自然数序列的代码：

```kotlin
fun main(args: Array<String>) {
   	val naturalNum = generateSequence(0) { it + 1 }
}
```

#### 序列相关操作

我们对集合操作的所有API都同样适用于序列，序列的好处就在于不会生成中间的集合。但通过上面的打印你也发现，我们打印出来的序列是一串很奇怪的字符，其实我们虽然不想产生中间的集合，但是我们希望最后产生的结果是一个集合，这里就涉及到几个关于序列的操作。

对于序列的操作我们分为中间操作和末端操作。

##### 序列的中间操作

所有对于序列进行处理变换的操作都是中间操作，比如`map`、`filter`等，对一个序列进行中间操作以后得到的是另一个序列，也就是说中间操作都是惰性的，就像下面的代码中演示的那样：

```kotlin
fun main(args: Array<String>) {
    println(personList.asSequence().map { it.name })

    val naturalNum = generateSequence(0) { it + 1 }
    println(naturalNum.takeWhile { it <= 100 })
}
```

上面的代码中`map`操作我们已经比较熟悉了，`takeWhile`这个操做我们可以从名字看出它是从一个序列中取出满足条件的一段子序列，上面代码中就是取出了0-100的子序列。

输出结果如下：

```
kotlin.sequences.TransformingSequence@87aac27
kotlin.sequences.TakeWhileSequence@4c873330
```

从输出结果看，中间操作的结果就是一个序列。

##### 序列的末端操作

序列的末端操作是用来对序列的一系列处理返回一个结果，返回的结果可能是集合、元素、数字、对象等。我们最常见的末端操作是`toList`。就像下面的代码那样：

```kotlin
fun main(args: Array<String>) {
    val naturalNum = generateSequence(0) { it + 1 }
    println(naturalNum.takeWhile { it <= 100 }.toList())
}
```

上面的代码使用`toList`来将子序列转化为一个List，并最终输出了0-100的整数。

由于序列是可迭代的，所以如果我们只是想输出元素，就可以直接使用迭代的方式输出数列中的元素，但是如果我们需要下标来随机访问元素，我们就需要首先将序列转为List了。就像下面这样：

```kotlin
fun main(args: Array<String>) {
    val naturalNum = generateSequence(0) { it + 1 }.takeWhile { it <= 100 }
    //迭代输出序列中的元素
    for (item in naturalNum){
        println(item)
    }
    val naturalList = naturalNum.toList()
    //下标随机访问List中的元素
    println(naturalList[10])
}
```

#### 及早求值和惰性求值

我们把常规的对List直接操作并产生中间集合的形式成为`及早求值`，对于序列的这种延迟求值操作的形式称为`惰性求值`。这里我们了解一下`及早求值`和`惰性求值`的区别。

我们以下面的代码为例，看看`及早求值`的处理方法：

```
fun main(args: Array<String>) {
    val numList = listOf(1, 2, 3, 4, 5, 6)
    numList.map { println("map($it)");it * it }
    .filter { println("filter($it)");it % 2 == 0 }
}
```

输出结果如下：

```kotlin
map(1)
map(2)
map(3)
map(4)
map(5)
map(6)
filter(1)
filter(4)
filter(9)
filter(16)
filter(25)
filter(36)
```

从输出可以看出，我们对List执行`及早求值`，它的执行方法是先对每个元素执行map操作，再对产生的中间集合中的每个元素执行filter操作。

我们看看下面这段`惰性求值`的代码：

```
fun main(args: Array<String>) {
   val numList = listOf(1, 2, 3, 4, 5, 6)
   numList.asSequence().map { println("map($it)");it * it }
   .filter { println("filter($it)");it % 2 == 0 }
}
```

你会发现上面的代码什么都没有输出，说明它对集合的处理操作确实被延期了，那怎样才能执行那些延期的操作呢？答案是使用`末端操作`函数，只有Kotlin发现我们需要最终结果的时候，延期的操作才会被执行，所以我们在上面的代码最后加上`toList`来运行看看。

```kotlin
fun main(args: Array<String>) {
    val numList = listOf(1, 2, 3, 4, 5, 6)
    numList.asSequence().map { println("map($it)");it * it }
    .filter { println("filter($it)");it % 2 == 0 }.toList()
}
```

输出结果如下：

```
map(1)
filter(1)
map(2)
filter(4)
map(3)
filter(9)
map(4)
filter(16)
map(5)
filter(25)
map(6)
filter(36)
```

是不是发现输出结果也有点出乎意料，从输出结果看出，`及早求值`是对先对集合执行完map操作再对中间结果执行filter操作，而序列的`惰性求值`是对每一个元素依次先执行map操作再执行filter操作。

可能通过上面的例子还看不出这种处理方式的好处，我们看下面一段代码：

```
fun main(args: Array<String>) {
    val numList = listOf(1, 2, 3, 4, 5, 6)
    numList.asSequence().map { println("map($it)");it * it }
            .find { println("find($it)");it >= 8 }
}
```

上面的代码是找出一个集合中元素的平方结果大于等于8的第一个值，我们看看输出结果：

```
map(1)
find(1)
map(2)
find(4)
map(3)
find(9)
```

从输出结果你会看出，`惰性求值`并没有将所有的元素都遍历处理一遍，因为处理到3的时候已经找到符合要求的值了，就结束了变换和寻找。如果对于`及早求值`，必须先完成集合中所有元素的map操作，`惰性求值`对于有大量数据的集合在性能上将会有一个较大的提升。

还有一个小技巧，这里顺便提一下，`先使用filter操作有助于减少变换的总次数`，这个简单点说就是先把符合要求的元素留下来，再继续后续的map操作，看看下面的代码就懂了。

```kotlin
fun main(args: Array<String>) {
    //先map后filter
    println(personList.asSequence().map(Person::name)
            .filter { it.length <= 2 }.toList())

    //先filter后map
    println(personList.asSequence()
            .filter { it.name.length <= 2}
            .map(Person::name).toList())
}
```

通过对比，你会发现，第二种先filter后map的处理方式直接将name长度大于2的元素不进行map操作，这样就会总体上减少了map的操作次数。而对于第一个中先map后filter的操作，肯定会先对每个元素进行map操作提取出其中的name。

#### 写在最后

Kotlin中的序列可以看作是Java8中流的翻版，但是Kotlin并没有实现序列在多个CPU上进行并行处理的能力，这一点需要注意。对于有大数据量的集合，序列无疑为我们提供了一种优雅的处理方式。