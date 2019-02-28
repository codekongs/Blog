> 在前面的文章中，我们已经看到了kotlin为了解决NPE问题作出的一些努力。这篇文章我们继续学习kotlin中与可空类型处理相关的一些知识。

#### 非空断言
在程序的编写过程中有这样一种场景，我们已经在前一个函数中对一个可空类型的变量进行了检查，之后我们在接下来的函数中使用这个变量，我们其实已经很明确地知道这个变量前面已经进行了判空处理，后续不可能为空，但是编译器无法清楚地推测出来，这时候在编译器眼里这个变量还是有可能为空，这就造成我们的代码中出现了很多无用的判空代码。

这种情况下我们就可以使用非断言“!!”。我们在一个可空的变量后面加上两个感叹号，用来明确告诉编译器我们确定这个变量是非空的。这样我们就简单粗暴地实现了将一个可空的变量转换为一个非空变量。

如下面的代码所示:
```
fun testNull(str: String?){
    val notNullStr: String = str!!
    println(notNullStr.length)
}
```
上面这种情况下相当于我们强制进行了转化，所以非空就由我们自己来保证，一旦传入了空类型，程序就会抛出异常。比如下面的异常:
```
Exception in thread "main" kotlin.KotlinNullPointerException
	at MainKt.testNull(Main.kt:6)
	at MainKt.main(Main.kt:2)
```
#### let函数
有时候，我们会有这样的需求，我们有一个可空类型的变量，但是我们要将其传入一个要求是非空参数的函数中，那我们必须在传递之前做非空判断，否则编译器不允许我们直接传入，就像下面这样:
```
fun sendMsg(msg: String){
    //xxxxx
}

fun main(args: Array<String>) {
    val msg: String? = "something"
    
    if (msg != null) sendMsg(msg)
}
```
针对上面这种很常见的情况，kotlin为我们提供了一个let函数，这个函数可以将调用它的参数转化为lambda表达式中的参数。就像下面这样:
```
msg.let {
    println(it)
}
```
上面lambda表达式中的it其实就是msg变量，结合上一节介绍过的安全调用运算符，我们可以这样写:
```
msg?.let{
    println(it.length)
}
```
上面的let函数在msg为空时不会发生调用，当msg不为空时，传递到let函数的lambda表达式中的it变量自然就变成了非空变量，这样就完美且优雅地将一个可空类型的变量转化为了非空类型的变量。
#### 延迟初始化属性
在kotlin中，你必须在构造函数中初始化所有非空类型的属性，否则就会报错，就像下面这样:
```
class Main{
    private var mText: String
    
    constructor(){
        mText = ""
    }

    fun showLen(){
        println(mText.length)
    }
}
```
但是有时候我们希望将属性的初始化放到自己的初始化函数中去完成，那这样我们就与kotlin中的规定冲突了，那我们就必须将属性声明为可空属性，但是那样，又导致我们在调用这个属性的方法时必须使用安全调用符或者是“!!”将可空类型转化为非空类型，那样的代码都不够简洁优雅，就像下面这样:
```
class Main{
    private var mText: String? = null

    fun initArgs(){
        mText = "xxxxx"
    }
    
    fun showLen(){
        println(mText?.length)
    }
}
```
对于这种我们想自己掌握属性的初始化，同时又想将属性声明为非空类型的情况，kotlin提供了延迟初始化属性，使用“lateinit”修饰符来表示一个延迟初始化的属性，拥有这个修饰符的属性，kotlin不会强制你必须在构造函数中初始化属性，你可以自己在任意的时刻自己掌握属性的初始化。就像下面这样:
```
class Main{
    private lateinit var mText: String

    fun initArgs(){
        mText = "xxxxx"
    }

    fun showLen(){
        println(mText.length)
    }
}
```
这样的代码就优雅了许多。假如我们上面属性的初始化时机不对，导致我们还没初始化这个属性就已经调用了这个属性的一些方法，就会报下面的这个异常:
```
Exception in thread "main" kotlin.UninitializedPropertyAccessException: 
lateinit property mText has not been initialized
```
异常也对这种情况说明得非常清楚，延迟初始化属性还没初始化便进行了访问。
#### 可空类型的扩展函数
在Java中，调用一个对象的方法，如果这个对象为null，就会发生NPE，然后kotlin中通过安全调用符来解决这个问题，保证了在变量不为null时调用才会发生。但是有时候我们就需要一些函数可以包括对null的检查，不需要我们在函数外部进行检查。
我们先看下面的这段Java代码:
```
public boolean isEmpty(String str){
    if (str == null){
        return true;
    }

    return str.isEmpty();
}
```
我们自定义的一个函数，在调用isEmpty()之前，我们必须自己先判空，处理掉为null这种情况，否则就可能出现NPE。我们想可以不可以定义一种扩展函数，它可以被null进行调用，并且不会报NPE，这就是kotlin中的可空类型的扩展函数。

注意：kotlin中只有扩展函数是可以针对可空类型的，常规的方法使用null去调用，要么是编译失败，要么就是NPE，我们下面举个例子，看下面的代码:
```
fun String?.isEmpty(): Boolean = this == null || this.isBlank()
```
这样一行代码，我们就为String的可空类型定义了一个isEmpty()这样一个扩展函数，也就是说null也可以调用这个函数。值得注意的是，函数中的this可能为null，第一个this进行了null的处理后，第二个this就成了一个非空类型的变量。然后我们就可以想下面这样进行调用了:
```
val nullStr: String? = null
println(nullStr.isEmpty())
```
注意，上面的调用我们并没有使用安全调用运算符，并且代码也没有报NPE，因为它是为String的可空类型的定义的扩展函数，在函数内部包含了对null值的处理。
#### 可空性与Java的统一
虽然kotlin中有可空类型和非空类型，但是在Java中却不是这样划分的，但是，我们会经常进行Java和kotlin的互调。那在互调时这种情况是怎么处理的呢?

##### 存在注解的情况
在Java或者是Android中都提供了这样两种注解“@NotNull”和“@Nullable”，一种表示变量不可为空，一种表示变量可为空，这主要是为了辅助编译器检查代码，就像下面这样:
```
public void showMsg(@NotNull String tag, @Nullable String msg){
        
}
```
当我们发生Java代码和kotlin代码的互相调用时，kotlin会将使用@Nullable注解的变量对应到kotlin中的可空类型的变量，将使用@NotNull注解的变量对应到kotlin中的非空类型的变量。
##### 不存在注解的情况
在我们日常使用的大部分类库中，其实都没有上面介绍的两种注解，那kotlin是怎么处理的呢?kotlin针对这种情况提出了一种类型叫平台类型，也就是没有注解的Java类型，对应为kotlin中的平台类型。你可以将其看作是kotlin中的可空类型，也可以看作是kotlin中的非空类型，而且你可以在其上面调用各种方法，编译器都不会报错，因为kotlin将变量的可空性控制权交到了使用者手上，完全由使用者来控制，而且如果你使用null进行了调用，就会报NPE。就像下面这样。
```
//我们在Java中定义的一个Message类
public class Message {
    private String msg;
    private int code;


    public Message(String msg, int code) {
        this.msg = msg;
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }
}
```

```
//在kotlin中进行调用
fun showMsg(msg: Message){
    println(msg.msg.toUpperCase())
}

fun main(args: Array<String>) {
    showMsg(Message(null, 500))
}
```
kotlin的编译器根本无法确定Message类中msg属性的可空性，所以只能将其作为平台类型由用户控制，所以上面的代码运行会出现NPE。

##### 继承的情况
当kotlin继承或者实现了Java中的某个类或者接口时，那对于方法的参数和返回值怎么处理呢?这里我们只讨论没有注解的平台类型，存在注解的情况跟前面说过的一样。我们看下面的代码例子:
```
//Java定义的接口
interface Account{
    void login(String username);
}
```

```
//kotlin中实现接口(参数类型可以是可空类型)
class UserAccount: Account{
    override fun login(username: String?) {

    }
}
```

```
//kotlin中实现接口(参数类型可以是不可空类型)
class UserAccount: Account{
    override fun login(username: String) {

    }
}
```
上面的两种情况，kotlin的编译器都不会报错，都可以编译通过。所以这个完全由开发者自己掌控。

##### kotlin中的类型在Java中被调用
前面说的几种情况都是Java中的类型在kotlin中的处理情况，那么kotlin中的可空类型和不可空类型在Java中是怎么处理的呢?
```
//kotlin中声明一个非空参数的函数
fun showToast(msg: String){
}
```
在Java中使用如下代码进行调用:
```
public static void main(String[] args) {
    MainKt.showToast(null);
}
```
然后你会发现抛出了一个异常如下:
```
Exception in thread "main" java.lang.IllegalArgumentException: 
Parameter specified as non-null is null: 
method MainKt.showToast, parameter msg
```
仔细想一下会发现有个疑问，虽然我们给showToast函数传递了一个null，但是我们的showToast函数中并没有对传入的参数进行使用啊。在Java中，只有我们在null对象上发生了调用，才会报异常，但是在koltin中则不同。再仔细查看上面的异常，你会发现它报的并不是NPE，而是参数异常。由此得出结论，kotlin中会为每一个声明为非空类型的参数生成一个非空断言，当我们尝试在Java中传入一个null值时，就会触发这个断言，也就是我们上面看到的异常。

#### 写在最后
本节主要介绍了kotlin中对于可空类型的处理的一些方法和运算符，可以看出kotlin中还是有很多特性来保证代码尽可能的优雅。但是在与Java进行互调的过程中，为了保证尽可能与Java良好的互调特性，所以存在上面介绍的平台类型，所以还是没法完全避免NPE的产生。