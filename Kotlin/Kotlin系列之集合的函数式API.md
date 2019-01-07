> 几乎各种语言都对集合操作提供了方便的库函数，Kotlin也不例外，今天就来说说Kotlin中与集合操作相关的函数式API。

在开始之前先说一下这里的演示数据，后面演示数据的代码将不再重复出现：

```kotlin
//数字集合
val numList = listOf(1, 2, 3, 4, 5, 6)

//People数据类
data class People(val name: String, val age: Int)

val peopleList = listOf(People("小红", 22),
        People("小明", 23),
        People("小白", 22))
```

#### filter和map过滤变换

`fliter`和`map`各种集合操作API的基础，如果你了解过Java8，应该对这两个函数不陌生，这里我们就不与Java8对比了，直接介绍他们在Kotlin中的用法。

`filter`是对一个集合中的所有元素应用一个表达式判断，然后返回一个判断结果为`true`的元素的新集合。说白了就是从一个集合中筛选出符合条件的元素并返回一个新集合。下面直接给出示例代码：

```kotlin
fun main(args: Array<String>) {
    //筛选出大于等于3的元素
    println(numList.filter { it >= 3 })

    //筛选出年龄等于22岁的人
    println(peopleList.filter { it.age == 22 })
}
```

如果对于代码中的Lambda表达式或者是数据类不了解的小伙伴，可以查看历史文章，这里就不再赘述了。

运行结果如下：

```
[3, 4, 5, 6]
[People(name=小红, age=22), People(name=小白, age=22)]
```

`map`是对一个集合中的每个元素应用一个表达式，然后生成一个新的集合，也是非常简单的，直接上代码：

```kotlin
fun main(args: Array<String>) {
    //求集合中每个元素的平方值
    println(numList.map { it * it })

    //提取出出集合中每个People对象的name属性
    println(peopleList.map { it.name })
}
```

运行结果如下：

```
[1, 4, 9, 16, 25, 36]
[小红, 小明, 小白]
```

当然上面的`filter`和`map`也可以链式调用，对一个集合做多次操作，比如我们想筛选出年龄等于22岁的人的姓名那就可以这样写：

```kotlin
fun main(args: Array<String>) {
    println(peopleList.filter { it.age == 22}.map { it.name })
}
```

输出结果如下：

```
[小红, 小白]
```

`filter`和`map`除了对list进行操作外，还可以操作map，注意这里的map是指key-value的map。看一下下面的代码分别对map的key和value进行过滤和变换操作。

```kotlin
fun main(args: Array<String>) {
    val numMap = mapOf("0" to 48, "1" to 49, "2" to 50, "3" to 51, "4" to 52)

    println(numMap.filter { it.key == "1" && it.value == 49 })
    println(numMap.filterKeys { it == "1" })
    println(numMap.filterValues { it == 48 })

    //返回一个list
    println(numMap.map { "key = " + it.key to "value = " + it.value})
    //返回一个新的map
    println(numMap.mapKeys { "key = " + it.key })
    //返回一个新的map
    println(numMap.mapValues { "value = " + it.value })
}
```

输出结果如下：

```
{1=49}
{1=49}
{0=48}
[(key = 0, value = 48), (key = 1, value = 49), (key = 2, value = 50), (key = 3, value = 51), (key = 4, value = 52)]
{key = 0=48, key = 1=49, key = 2=50, key = 3=51, key = 4=52}
{0=value = 48, 1=value = 49, 2=value = 50, 3=value = 51, 4=value = 52}
```

#### all、any、count和find查找

`all`、`any`、`count`和`find`都是和查找相关的API。

`all`用来确定一个集合中的元素是否都满足某个条件，而`any`是判断集合中是否有元素满足某个条件。下面是示例代码：

```kotlin
fun main(args: Array<String>) {
    //判断是否所有元素都小于6
    println(numList.all { it <= 6 })

    //判断是否所有人的年龄都小于25岁
    println(peopleList.all { it.age < 25 })
    
    //判断是否存在大于100的数
    println(numList.any { it > 10 })

    //判断是否存在年龄大于25岁的人
    println(peopleList.any{ it.age > 25})
}
```

输出结果如下：

```
true
true
false
false
```

`count`用于对我们上面的判断检查进行统计，统计有多少个元素是满足条件的。就像下面这样：

```kotlin
fun main(args: Array<String>) {
    //统计集合中小于等于3的元素的个数
    println(numList.count { it <= 3 })

    //统计集合中年龄小于等于22的人的个数
    println(peopleList.count { it.age <= 22 })
}
```

输出结果如下：

```
3
2
```

其实上面的需求我们还有一种方式可以实现，就是先过滤出满足条件的元素组成一个新的集合，再获取这个新的集合的大小，就像下面这样：

```kotlin
fun main(args: Array<String>) {
    println(numList.filter { it <= 3 }.size)

    println(peopleList.filter { it.age <= 22 }.size)
}
```

上面的代码虽然可以完成我们的需求，但是它相较于前一种`count`的方式，生成了一个新的集合，其实我们根本不关心元素本身，所以`count`方式更加高效。

#### groupBy分组

`groupBy`用于对元素进行分组，分组后的结果是一个Map，它的类型是这样的Map<key, List<T>>，如果你了解SQL，一定对`groupBy`不陌生。下面看看我们根据年龄对一个集合进行分组的代码示例：

```kotlin
fun main(args: Array<String>) {
    println(peopleList.groupBy { it.age })
}
```

输出结果如下：

```
{22=[People(name=小红, age=22), People(name=小白, age=22)], 23=[People(name=小明, age=23)]}
```

可以看出它的输出结果是`Map<Int, List<People>>`类型。

#### flatMap和flatten铺平

`flatMap`和`flatten`的都是对集合的操作，你可以把它们简单理解为对集合元素进行展开再合并到一起形成一个新的集合。

其中`flatMap`是对一个集合进行map变换以后，再将变化后的多个集合合并为一个集合。如果你只是想单纯地合并多个集合而不需要变换，那就选用`flatten`，下面是具体的示例代码：

```kotlin
fun main(args: Array<String>) {
    val strList = listOf("abc", "def", "hij")
    println(strList.flatMap { it.toList() })

    val lists = listOf(listOf("abc", "bnm"), listOf("acv", "wnm"), listOf("awm"))
    println(lists.flatten())
}
```

输出结果如下：

```
[a, b, c, d, e, f, h, i, j]
[abc, bnm, acv, wnm]
```

是不是发现比我们手动合并多个集合方便多了。

#### 写在最后

通过对Kotlin中集合操作API的介绍，又一次领略到Kotlin的简洁，以后遇到集合处理相关的需求，应该首先看看Kotlin是否已经提供了相关的API，然后选用合适的API进行处理即可。