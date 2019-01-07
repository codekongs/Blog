> 今天来看看Kotlin中关于lambda的两个函数with和apply，我们将其称为带接收者的lambda，不了解为什么会这么命令，看完下面的实例你应该就可以理解了。

注意，上面也已经说了，with和apply其实是两个函数，虽然它们看起来像是关键字。

#### with函数

##### 简单使用
我们在Java中经常会写下面这样的代码：

```java
public String getRes() {
    StringBuilder result = new StringBuilder();
    result.append("start\n");
    for (int i = 'A'; i < 'Z'; i++) {
        result.append((char)i);
    }

    result.append("\nover");
    return result.toString();
}
```
我们上面这所以可以反复调用result的append()方法，是因为StringBuilder每次调用append()方法都会返回这个对象本身。

Kotlin中认为这样连续调用同一个对象的不同方法，每次都需要显式写出result这个对象的方法不够简介，所以提供了result方法来简化这个过程，上面的代码改造之后，在Kotlin中可以这样书写：

```kotlin
fun getRes(): String{
    val result = StringBuilder()
    return with(result){

        //第一种: 显式指定对象
        result.append("start\n")

        for (i in 'A'..'Z'){

            //第二种: 省略对象
            append(i)
        }

        append("\nover")
        //第三种: 使用this来表示result对象
        this.toString()
    }
}
```
上面的代码虽然短小，但包含着非常丰富的信息，我们一步步分析一下：
1. 定义的函数getRes并指定返回值为String
2. 上面说了with是一个函数，其实它有两个参数，第一个参数是需要传入的对象，第二个参数是一个lambda表达式，根据lambda表达式的语法规则，这个lambda表达式可以写在括号外面。with函数可以让在这个lambda表达式中，所有涉及到对result对象操作的函数都可以省略调用对象result而直接调用其函数。
3. 上面代码中我演示了在with的lambda表达式中的三种调用方法。一种是显式指定对象，一种是直接省略对象，一种是使用this调用。这里的this就表示with函数中传入的对象。
4. 最后一行的结果就会作为这个with函数调用的返回值。

##### 进阶使用
你看到上面的写法已经很简洁了，其实它还可以更简洁。因为在kotlin中，最后一个表达式的值，就是整个函数的返回值，同时，我们上面也说过了，在with函数的lambda表达式中，我们可以直接调用传入对象的函数，所以，上面的代码，又有了下面更加简洁的写法：
```kotlin
fun getRes(): String{
    return with(StringBuilder()){
        for (i in 'A'..'Z'){
            append(i)
        }

        append("\nover")
        toString()
    }
}
```
是不是看起来简洁了许多，其实就是把一些可以简化的步骤组合到了一起，为什么可以这么写，前面其实已经说过了。
##### 可能存在的问题
上面我们的简洁写法存在一个问题，接入我们在lambda中调用的一个方法，与StringBuilder中的方法重名了，我们该怎么办呢？就像下面这样：
```kotlin
class Sample{
    fun append(str: String){
        str + "\n"
    }

    fun getRes(): String{
        return with(StringBuilder()){
            for (i in 'A'..'Z'){
                append(i)
            }

            this@Sample.append("\nover")
            toString()
        }
    }
}
```
上面的示例代码中，我们的Sample类中已经定义了一个append方法，但是我们传入的with函数的StringBuilder中也存在一个同名的append方法。这时候，如果我们只是单纯使用append调用，只会调用StringBuilder的append方法，如果我们想调用我们自己定义的append方法，那就可以像示例代码中一样，使用this@外部类.方法名这种调用方式。

#### apply函数
apply函数的用法和with函数的用法几乎一模一样，唯一的差异在于，我们在使用with函数时，最终的返回结果是我们在lambda表达式中最后一个表达式的值，比如上面的例子就是toString的结果作为整个with函数的返回值。
而apply则不同，它的返回值是我们传入的对象。可能听起来有点抽象，我们这次用apply来重写上面那个例子，代码如下：
```kotlin
fun getRes(): String{
    return StringBuilder().apply {
        for (i in 'A'..'Z'){
            append(i)
        }
        append("\nover")
    }.toString()
}
```
这里注意，我们不是在apply函数中传入我们的StringBuilder对象，而是直接在我们的StringBuilder对象上调用apply函数，这是因为apply是一个扩展函数，我们可以在任意对象上调用它。
然后你会注意到，我们在lambda表达式的外部调用了toString方法，正如前面所说，我们的apply函数返回的是我们传入的对象，即返回了StringBuilder对象。

apply函数的出现，成功让我们的代码在整洁的同时，保持了代码的紧凑。
其实你看到上面的例子，其实我们在Java中对于构建一个特定属性的对象，我们通常是使用Builder模式来做，但是在kotlin中，我们只需要借助apply函数就可以随意定制任意对象。
#### 写在最后
回到一开始的疑问，为什么叫做带接收者的lambda，就是无论是我们传递给with还是apply函数的对象，都可以在lambda内部自由调用这个对象的方法，这在Java里面是没有这种能力的，这里我们传递给with或者apply的那个对象就是我们所谓的**接收者**。