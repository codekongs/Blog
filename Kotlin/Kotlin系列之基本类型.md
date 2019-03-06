> 今天一起来看看kotlin中的基本类型，包括基本的数据类型和其他一些特殊的与Java不同的类型。

#### 基本数据类型
在Java中数据类型被分为基本数据类型和引用数据类型。在kotlin中所有的数据类型都是引用数据类型。与Java中的数据类型对应，kotlin中的数据类型有如下几种:

|数据类型|java中的类型|kotlin中的类型|
| -- | -- | -- |
|整数|byte short int long|Byte Short Int Long|
|浮点数|float double|Float Double|
|字符|char|Char|
|布尔|boolean|Boolean|

当然上面没有列出Java对应的包装类型。你会发现kotlin中的类型就是Java中的类型的首字母大写，更像是Java中的包装类型。

在kotlin中不区分基本数据类型和包装类型，它永远都只有一种类型。也就是在kotlin程序中，所有的类型都是对象类型，这样看起来kotlin是一门比Java还更纯粹的面向对象语言。

koltin之所以这样设计，是可以允许我们可以对数字类型的值直接调用方法，就像下面这样:

```
fun main(args: Array<String>) {
    12.toString()
}
```

这在Java中是没法做到的。

但是我们都知道，Java中的包装类型是会损耗一定的性能的，那kotlin中的所有类型都是对象类型，会不会影响从程序性能呢?

我们都知道kotlin程序最终是被编译成class文件在JVM上运行，所以，尽管我们在kotlin程序中使用的是对象类型，但是在编译时kotlin会尽可能编译成Java中的基本数据类型，使程序运行更加高效。

下面我们写个简单的程序验证一下，有如下的kotlin函数，它被写在Main.kt这个文件中，它的是Int:
```
fun test(num: Int){
   //...
}
```
我们在IDEA里面找到编译后的class文件，使用下面的命令进行反编译:
```
javap -c MainKt
```
得到下面的结果:
```
Compiled from "Main.kt"
public final class MainKt {
  public static final void test(int);
}
```
你会看到从class文件反编译出的代码中函数的参数已经变成了int类型。

下面的两种情况下kotlin不会将数据类型编译成Java的基本数据类型。

##### 1.泛型类

在Java中，受限于Java实现泛型的方式，JVM不支持将基本数据类型作为泛型的类型参数，泛型集合中是不能存储基本数据类型的，我们是不可能写出`List<int>`这样的程序的集合的，我们只能写出`List<Integer>`这样的代码。

由于kotlin代码最终也是被编译成class文件跑在JVM上，所以在kotlin中，作为泛型的类型参数，kotlin中的类型会被编译成包装类型。

大家也可以使用上面掩演示过的反编译class文件的方式进行验证。

##### 2.可空的基本数据类型
还有一种情况是可空的数据类型，在kotlin中也会被编译成对应的包装类型，这个其实很好理解，因为可空类型是要被允许存储null值的，而基本数据类型是没法存储null值的。

就像下面的代码，在kotlin中会被编译成对应的包装类型:
```
fun test(num1: Int?, num2: Int?): Boolean{
    if (num1 == null || num2 == null){
        return false
    }
    return num1 > num2
}
```
上面的函数的两个参数都会被编译成Interger类型，但是函数返回值Boolean会被编译为bolean类型，下面是反编译class文件后的内容，只给出了函数签名相关的编译结果:
```
Compiled from "Main.kt"
public final class MainKt {
  public static final boolean test(java.lang.Integer, java.lang.Integer);
}
```
同时你还应该注意到，因为是可空类型，两个Int值在比较之前必须先判空，kotlin才运行之后的比较运算。
#### 数据类型转换函数
在Java中，一些数据类型是可以自动像另一些数据类型转化的。比如当将一个int类型的变量赋值给long类型的变量时，就会自动发生转化。

但是在kotlin中，是不允许这种自动转化的。kotlin为每一种数据类型都提供了转化函数。当你将一种数据类型赋值给另一种数据类型的变量时，必须显式调用转换函数，就像下面的代码那样:

```
fun main(args: Array<String>) {
    val intNum: Int = 13
    val longNum: Long = intNum.toLong()
    val intNum2: Int = longNum.toInt()
}
```
无论是从大范围的数转换为小范围的数，还是小范围的数转换为大范围的数，都必须显式调用转换函数，当将一个大范围的数转换为小范围的数时会发生截断，这种情况跟我们java中强行将一个大范围的数转换为一个小范围的数的处理方法是一致的。
#### Any类型
在kotlin中，Any类型是所有非空类型的根类型，相应的Any?类型是所有可空类型的根类型。Any类型相当于Java中的Object类型。

在与Java进行互操作时，Java中的Object类型会被当成kotlin中的Any类型。前面几节的内容也提到了平台类型，其实准确来说，Java中的Object类型作为方法返回值或者是方法参数，在Kotlin中其实是被当作平台类型，也就是可以当作可空类型，也可以当作非空类型。除非他们被@Nullable或者是@NotNull这些注解修饰，才会在kotlin中确定为具体的类型。

相应的kotlin中的Any类型在底层被编译为Java的Object类型。如下面的简单的方法:
```
fun get(num: Any){

}
```
被反编译后的方法签名如下:
```
public static final void get(java.lang.Object);
```
#### Unit类型
在kotlin中，Unit类型相当于Java中的void，但是又不仅仅是void。先说说第一种，相当于Java中的void，我们看下面这个方法的定义:
```
fun after(): Unit{
    println(".....")
}
```
当然上面的代码Unit是可以省略的，这里写出来就是给大家看看它的第一种用法，我们之前所写的没有返回值的函数，其实返回值都是Unit，只是他们都被省略了而已。

第二种情况，Unit在kotlin中是一种类型，比如String也是一种类型。但是比较特殊的是，在kotlin中只有一个值是Unit类型，它也叫Unit类型，是不是有点绕，看下面的代码:
```
val str1: String = "str1"
val  str2: String = "str2"
```
比如上面的代码中，”str1“和“str2”这两个字符串值都是String类型的，但是在kotlin中Unit类型的值只有一个，它叫Unit，看下面的代码:
```
val x: Unit = Unit
```
上面x是Unit类型，你没法给它赋值其它值，只能赋值为Unit。
那kotlin中为什么要这么做呢？其实是为了实现一种优雅的泛型参数的处理。

我们假设一种场景，我们想定义一个接口，它里面的一个函数的返回值返回一个泛型参数，我们先看下面Java的实现方式:
```
//接口定义
public interface IHandler<T> {
    public T getSomething();
}

//接口实现1,方法返回String类型
public class Handler1 implements IHandler<String> {

    @Override
    public String getSomething() {
        String str = "";
        return str;
    }
}

//接口实现2,方法返回Void类型
public class Handler2 implements IHandler<Void> {

    @Override
    public Void getSomething() {
        return null;
    }
}
```
上面，我们通过传入不同的泛型参数，来实现函数返回不同的返回值，但是我们看第二种接口实现，我们想让函数返回void，我们必须使用Void类型，注意这是个不可实例化的对象类型，因此我们必须在函数末尾写上return null，这就让代码不够优雅。如果我们想让函数返回void类型，那么我们就必须为返回void这种特殊的情况重新定义一个和原来接口一样的接口，只是函数的返回值写死为void，比如下面这样:
```
public interface IHandler2 {
    void getSomething();
}
```

那我们看看我们在kotlin中如何优雅地实现上面的接口:
```
class KtHandler: IHandler<Unit>{
    override fun getSomething() {
        
    }
}
```
看看上面的代码，如果我们将其补充完整是下面这样:
```
class KtHandler: IHandler<Unit>{
    override fun getSomething(): Unit{
        return Unit
    }
}
```
但是由于一个方法的返回值为Unit是可以省略的，同时，在一个没有返回值的方法的最后一行，kotlin会隐式返回Unit，所以就变成了我们前面看到的那样。也没有多余的return null，也不需要为没有返回值的函数这种特殊情况单独处理，kotlin很好地兼容了这两种情况，是不是超级优雅。
#### Nothing类型
Nothing是kotlin中一种特殊的类型，它只能作为函数返回值，或者作为泛型函数的类型参数，它表示一个函数永远不会正常结束。什么意思呢？比如一个无限循环的函数就可以声明返回值为Nothing，或者说一个函数一定会抛出异常，那他们的返回值就可以声明为Noting，就像下面这样:
```
fun loop(str: String): Nothing{
    while (true){
        
    }
}

fun fail(str: String): Nothing{
    throw IllegalArgumentException(str)
}
```
#### 写在最后
这一节我们主要说了kotlin中的一些基本类型，一些类型与Java有些相似，也有一些是新增的类型，但是为了良好的互操作性，基本在Java都有对应的类型，了解了这些类型，相信会让我们写出的代码更加优雅合理。