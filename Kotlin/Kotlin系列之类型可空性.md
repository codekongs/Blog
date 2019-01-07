作为一个程序员，最常见的问题恐怕就是NPE了吧，有时候即使很小心的编码，还是避免不了出现NPE，在Kotlin中，它力争把这个在运行时常常出现的问题在编译器解决掉，让我们写出更加健壮不易崩溃的代码。

#### Java的类型系统存在的问题
在说kotlin之前，我们先谈谈Java中的类型系统。什么是类型呢？通俗来讲其实就是对所有我们要表示的数据确定一个具体的分类。
比如，我们把12这个数据分为int这个分类(或类型)，我们把"ABC"这个数据分为String分类(或类型)。那为什么要分类呢？其实是为了对不同的类型做不同的运算和处理。比如知道一个数据是int类型，那我们就可以对其进行加减乘除等运算，知道一个数据是字符串类型，我们就可以对其求长度。
但是，在Java的类型系统中存在这样的问题，对于所有的引用类型的对象，当它们没有被初始化时，他们的值都为null。这里存在什么问题呢。看下面这行简单的Java代码:
```java
String str = "ABC";
System.out.println(str.length());
str = null;
System.out.println(str.length());
```
上面的代码，我们声明了一个String类型的变量str，然后我们将"ABC"这个字符串赋值给它，我们调用length()函数，它可以正确打印出字符串长度，这是很合理的。但当我给str赋值为null后，我们同样可以对str变量调用length()函数，但是这时候却报了NPE。这时候我们回头分析一下这段我们习以为常的代码存在的问题，那就是在Java的类型系统中，所有的引用类型都可以允许被赋值为null，而且Java也允许我们对null调用length()方法，至少这么写是可以正常编译的，但是却在运行时报NPE。为了避免NPE我们不得不在调用方法前对str变量进行额外的null检查，这就导致我们的代码中不得不出现很多非空检查的防御性代码。

下面就来看看kotlin的设计者是如何解决上述问题的。
#### kotlin中的可空类型
上面我们说，Java存在的问题是对于一个引用类型的变量，它允许我们将引用类型对象和null都可以赋值给这个变量。

kotlin的做法也比较简单，你声明的变量默认就只能将非空的值赋值给这个变量，一旦你将null赋值给这个变量，你会发现你的代码连编译都过不了。这样就在一定程度上在编译期提前帮我我们暴露了问题。
有时候你想声明一个变量可以给它赋值为null，那你就必须显式声明它是可空类型，也就表明你很清楚它赋值为null是没问题的。

下面的代码给出非空类型的声明方式。
```kotlin
//str变量为非空类型
fun len(str: String) = str.length

//传入null编译会报错
fun main(args: Array<String>) {
    len(null)
}
```

如果你想让一个变量允许赋值为null，即将其变为可空类型，声明方式为在类型后面加一个“?”，如下代码所示:
```kotlin
//str变量为可空类型
fun len(str: String?) = str.length

//传入null编译不会报错
fun main(args: Array<String>) {
    len(null)
}
```
下面是可空类型和非空类型的几点特性，都很好理解，结合下面的示例代码:
```kotlin
fun main(args: Array<String>) {
    var str1: String? = null
    //可空类型是不允许直接调用String的方法和属性的,编译报错
    val len = str1.length

    //可空类型变量是不可以直接赋值给非空类型变量的,编译报错
    var str2: String = str1
    
    //非空类型变量是可以直接赋值给可空类型变量的
    var str3: String = "value"
    str1 = str3
}
```
- 可空类型是不允许直接调用对象的方法和属性的，编译报错
- 可空类型变量是不可以直接赋值给非空类型变量的，编译报错
- 非空类型变量是可以直接赋值给可空类型变量的

还有一点要注意，对于一个可空类型，如果你显式做了null的检查，那你就可以直接在可空类型上面调用这个对象的方法和属性了，如下面的代码:
```kotlin
fun len(str: String?): Int = 
    if (str != null) str.length else 0
```
在没有判断null之前，调用str.length，编译器会报错，现在判断完null之后，编译器就允许我们调用了。
#### 安全调用运算符
在Java中，我们经常会写下面的代码:
```java
if (str != null){
    return str.toUpperCase();
}else {
    return null;
}
```
在kotlin中，我们对于可空类型的变量，也会写出下面的代码:
```kotlin
if (str != null) str.toUpperCase() else return null
```
kotlin中对于这种常见的模版代码，提供了一种专门的运算符，叫做安全调用运算符，它的具体形式是"?."
这个运算符的运算规则是，如果被调用的变量不是null，则正常调用并返回结果，如果调用的变量等于null，则这次调用不会发生，直接返回null。这就有效避免了NPE问题。
我们看看上面的例子，在kotlin中使用安全调用运算符该怎么写:
```kotlin
return str?.toUpperCase()
```
对，就是这么简单的一行，就把判空操作直接也包含在内了。
注意，安全调用运算符的返回结果是一个可空类型的变量，因为它有可能会返回null。即像下面这样:
```kotlin
val res: String? = str?.toUpperCase();
```
上面的例子可能还不够吸引你，下面的这个例子绝对可以让你喜欢上这个运算符:
我们定义一个对象的时候，经常会有对象嵌套的情况，但是当我们从一个对象里面访问嵌套的对象的属性时，经常需要做很多层的null检查，就像下面的kotlin代码一样:
```kotlin
data class Son(val name: String?, val age: Int?)

data class Person(val name: String?, val age: Int?, val son: Son?)

fun Person.getSonAge(): Int{
    if (son != null){
        val sonAge = son.age
        return if (sonAge != null) sonAge else 0
    }else{
        return 0
    }
}
```
对于上面的例子，我们发现每个变量都有可能是null，我们就必须在访问它们的属性时必须加上丑陋的null检查代码，否则就有可能造成NPE，当我们使用安全调用运算符后，我们就可以像下面这样一行调用了:
```kotlin
fun Person.getSonAge(): Int? = son?.age
```
无论我们的对象定义的嵌套层级有多深，我们都可以使用安全调用运算符直接访问属性，当其中某一层的属性为null时，就直接返回null，而不再去访问后面的属性，使代码简洁了很多。
#### null合并运算符
上面的代码还是存在不太优雅的地方。上面的例子，当对象属性为空时，我们希望age可以返回0，而不是null，在Java中我们可以使用三目运算符，在kotlin中不存在三目运算符，那我们就必须使用if判断了，像下面这样写:
```kotlin
fun Person.getSonAge() = if (son?.age != null) son.age else 0
```
但是kotlin中提供了null合并运算符"?:"，可以简化上面的判断逻辑，根据名字就知道，它是将我们对null的判断进行了合并简化，注意，这个运算符跟Java的三目运算符很像，但它是个二元运算符。
它的运算规则是，如果第一个运算数不为null，那结果就是第一个运算数，如果第一个运算数为null，那结果就是第二个运算数，所以上面的代码可以简化为下面的样子:
```kotlin
fun Person.getSonAge() = son?.age ?: 0
```
是不是简洁了很多，但是个人感觉第一次接触还是觉得不利于代码的理解，熟悉了以后还是很不错的。
当然，我们有时候不想在值为null的时候返回一个默认值，我们想抛出一个异常，那也是可以的，就像下面这样:
```kotlin
fun Person.getSonAge() = son?.age ?: throw IllegalArgumentException("no age")
```
#### 安全转换运算符
在Java中，我们想验证一个对象是不是某个类型，是使用instanceof关键字来检查，然后再进行对象的强转，如果我们直接强转就会出现ClassCastException异常。在kotlin中，同样的功能我们使用is进行类型检查，使用as进行类型强转，就像下面这样:
```kotlin
class Person(val name: String, val age: Int){
    override fun equals(other: Any?): Boolean {
        if (other is Person){
            //检查成功后会只能转化other对象为Person类型
            // val otherPerson = other as Person
            return other.name == name && other.age == age
        }else{
            return false
        }
    }

    override fun hashCode(): Int = name.hashCode() * 37 + age
}
```
上面的例子是重写了Person类的equals和hashCode方法。
kotlin针对上面的情况，为我们提供了安全转化运算符"as?"，简单来说就是如果一个对象可以成功转换为某个类型，则会自动转换，否则返回null，这样我们再利用null合并运算符进行处理，就可以写出非常优雅的代码，下面的代码用更优雅的方式实现了上面的功能:
```kotlin
class Person(val name: String, val age: Int){
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false;
        return other.name == name && other.age == age
    }

    override fun hashCode(): Int = name.hashCode() * 37 + age
}
```
是不是非常简洁，当然还是上面说的，初次接触会觉得这个东西对代码可读性不太友好，使用熟悉以后就写起来非常舒服了。
#### 写在最后
相对于Java，kotlin中对类型系统中的可空性做了改进，并通过上面一系列的特性和运算符尽可能减少NPE问题的出现，帮我我们写出安全的代码，同时也不失其简洁性。