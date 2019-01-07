>今天一起来看看Kotlin中与Lambda相关的成员引用的内容。

#### 定义
先说说什么是成员引用，这里的引用指的就是变量引用，就相当于Java中的引用概念。成员的概念这里包含了成员变量和成员方法。这都是很官方的的说法。说白了，就是类里面的变量和函数。所以我们这篇文章讨论的就是如果一个成员函数或者变量已经被定义好了，我们如何使用Lambda表达式的语法，将它传递给另一个函数。
#### 成员引用语法
我们的成员引用遵循下面的语法。
```
类名::成员变量名
类名::成员方法名
```
这里通过`::`这个运算符负责将一个成员转化成一个值。
##### 一般类的成员引用
我们按照上面的语法来写一个一般类的成员引用的例子
```
class Person{
    var age = 0

    fun doSomething(){
        println("do something")
    }
}

fun main(args: Array<String>) {
    val age = Person::age
    val doFun = Person::doSomething

    val p = Person()
    doFun(p)
    println(age(p))
}
```
上面的例子虽然有点繁琐，但重点是为了演示这种用法。而且要注意一点，上面的成员方法的引用`::`之后只有方法名，没有`()`。
当然成员引用也可以写成下面这样，这样看起来更清晰好理解一点。
```
fun main(args: Array<String>) {
    val p = Person()
    val getAge = {p1: Person -> p1.age}
    getAge(p)
}
```
##### 引用顶层函数
上面的引用都是要依托于类名，但是顶层函数是不依附于某个类的，那在Lambda表达式中该怎样引用呢？
```
//顶层函数
fun  printInfo() = println("Info")

fun main(args: Array<String>) {
    val printFun = ::printInfo
}
```
可以看到引用方式是直接省略了类名，通过`::方法名`就可以引用了。
##### 引用构造函数
下面在说说Lambda中如何引用构造函数，我们知道我们调用构造函数就创建了一个对象，那如果我把构造函数的引用赋值给一个变量，就可以对构造函数进行存储和传递，然后在需要创建对象的时候再去创建。
```
//数据类
data class User(val name: String, val age: Int)
fun main(args: Array<String>) {
    val createUser = ::User
    val user = createUser("cmq", 12)
}
```
它的使用方式为`::类名`，跟顶层函数的应用有点像，看上面的例子就好像给构造函数起了个别名，但它却是可以被传递的。
##### 引用扩展函数
我们在前面还说过扩展函数，那么在Lambda中如何引用扩展函数呢。其实扩展函数可以看作是被扩展的类的成员函数，所以引用方式和一般类的成员引用是一样的。
```
fun User.isOld() = age > 60

fun main(args: Array<String>) {
  val siOldFun = User::isOld
}
```
#### 写在最后
今天的内容就结束了，总结为一句话就是我们知道了如何将成员函数和属性变成可以传递的参数，上面的内容基本覆盖了常用的场景，又一次看到了Kotlin的简洁。
