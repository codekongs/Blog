> 上一篇文章讲到了最基本的Lambda表达式，今天这篇文章继续讲Lambda表达式中的在作用域中访问变量。

##### Java中的内部类访问变量
当我们在函数内部使用匿名内部类时，我们可以在匿名内部类内使用函数的参数和函数内的局部变量。当我们在使用Lambda表达式时，我们也可以访问这个函数的参数和使用那些在Lambda表达式之前定义的变量。

下面先看一个在Java中匿名内部类中访问函数参数和局部变量的例子。
```
public void search() {
    final String str = "xxxx";
    new Thread(new Runnable() {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                System.out.println(str);
            }
        }
    }).start();
}
```
上面的这个例子中局部变量**str**在匿名内部类中被使用，所以必须要加final修饰符。

如果你使用的JDK是Java8版本，那你会发现你不写final修饰符也是不会报错的。这因为Java8中final修饰符不是必需的，但是如果你尝试在内部类中更改str的值，IDE就会告诉你，在匿名内部类中使用的局部变量是final的，不可以被修改。所以在Java8中如果你没有尝试去修改那个值，final修饰符是可以省略的。

那为什么在Java中匿名内部类中使用的变量必须是final类型呢？这里涉及到一个作用域的问题。str多的作用域是search()这个函数，函数运行结束，那这个局部变量就消失了，但是我们的匿名内部类的执行时机并不是在search()函数执行完成前，可能search()函数已经结束了，匿名内部类线程才会执行，如果这时候str不是final类型，它就已经被回收了，当线程类执行的时候就会找不到这个变量而报错。如果使用final修饰符，Java就会复制一份这个变量作为内部类的成员变量，由于是final修饰的，还保证了这份复制过来的变量不会被篡改，使内部类和外部类的变量保持一致性。

##### Kotlin在作用域中访问变量
上面解释了Java中匿名内部类中变量访问的情况及原理，其实把上面的匿名内部类换成Lambda表达式也可以做同样的事，达到同样的效果。
那下面看看在Kotlin中使用Lambda表达式访问作用域中的变量的规则。

在Kotlin中在Lambda表达式内部可以访问外部的变量，而无需声明为final，而且在Lambda内部可以修改这些变量。下面看一个示例

```
fun countThings(datas : List<Int>){
    var count = 0
    datas.forEach {
        if (it == 0){
            count++
        }
    }
    print("count = $count")
}
```
上面代码forEach里面传递的就是一个Lambda表达式，对于局部变量count并没有被声明为final，同时这个变量还可以在Lambda表达式内部被修改。

与Java比起来这是不是有点不可思议，那Kotlin是怎么做到的。

在分析之前我们先来看一下，加入我们在Java中是如何实现匿名内部类中修改外部变量的值的。
```
public void search() {
    final String[] str = {"xxxx"};
    new Thread(() -> {
        for (int i = 0; i < 10; i++) {
            str[0] = str[0] + "|" + i;
            System.out.println(str[0]);
        }
    }).start();
}
```
上面的代码中Java为了让我们让匿名内部类可以修改外部的变量，我们创建了一个单元素的数组，并声明为final类型。这样虽然str数组是final的，但它其中的元素却是可以修改的，这样就既保证了匿名内部类从函数中复制到匿名内部类内部的变量是不可变的，又保证了我们可以在匿名内部类中修改这个变量值。

Java除了使用上面的方法解决这个问题外，还有一种方法如下
```
public void search() {
    final Ref<String> ref = new Ref<>("xxxx");
    new Thread(() -> {
        for (int i = 0; i < 10; i++) {
            ref.value= ref.value + "|" + i;
            System.out.println(ref.value);
        }
    }).start();
}


static class Ref<T>{
    T value;
    public  Ref(T value){
        this.value = value;
    }
}
```
上面的代码中我们使用了一个静态内部类，虽然静态内部类的引用是final类型的，但我们却可以修改它内部的value属性，也就达到了在Lambda表达式中修改外部变量的目的。

其实上面的这种方法，正是Kotlin使用的方法。Kotlin内部就是通过这样的方法使我们可以在Lambda内部修改外部的变量。只是不需要我们显式去创建这样的包装器类。

所以用专业一点的术语解释就是，默认情况下，局部变量的声明周期是被限制在了声明这个变量的函数中，但是如果它在Lambda内部被使用了，我们就称它被Lambda捕获了，这时候，使用这个变量的代码就会被存储并稍后执行。当捕获了一个final变量时，它的值就会和使用这个值的Lambda表达式一起被存储，如果对非final变量，它的值就像上面演示的一样被封装在一个包装器内部，这样这个值就可以被改变，同时会被这个包装器类的引用和Lambda代码一起存储。

说白了，就是如果你只是在Lambda内部使用这个值而不修改它，那就复制一份它的值到Lambda表达式内部，如果你在Lambda内部使用它，那Kotlin就会创建一个包装器类，同时把这个包装器类的引用复制到Lambda表达式内部方便你修改它。

##### 写在最后
本节涉及到的这个只是点还是蛮重要的，但是Kotlin已经帮我们隐藏了背后的实现细节，但我觉得背后的原因还是有必要搞清楚的。
