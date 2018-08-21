#Java和Kotlin之间的相互调用

在Kotlin中可以直接调用Java，在Java中调用Kotlin，需要在Kotlin文件中添加相关注解。

1. Kotlin文件中的方法自动被变成静态的方法，默认调用就是文件名+Kt.方法名，例如：UtilsKt.echo()。假如你想修改类名那么你可以在Kotlin文件中使用注解'@file:JvmName("Utils")'，那么调用就可以直接Utils.echo()。
2. 在Kotlin中暴露属性可以使用注解‘@JvmFiled’
3. 在Kotlin中暴露静态方法可以使用注解'@JvmStatic'
4. Kotlin扩展函数假如被Java调用那么函数的第一个参数是对象本身，并且不能省略默认参数。

#####这是Java文件代码

```
public class JavaMain {

    //in在Kotlin中是关键字,在Kotlin中调用需要使用``进行转义
    public static String in = "IN";

    public static void main(String[] args) {

        MainKotlin.echo("叶龙");
        MainKotlin.testClass(JavaMain.class);

        KotlinMain main = new KotlinMain();
        main.id = 13;
        main.getName();

        Singleton.INSTANCE.sayMessage("叶龙");
        Singleton.printName("叶龙");

        Singleton.provider = new Provider("叶龙");
        Singleton.provider.setName("Longe666");
    }

    public void sayHello(String name) {
        System.out.println("你好，" + name);
    }

    public static int number() {
        return 28;
    }

}
```

#####这是Kotlin代码

```
@file:JvmName("MainKotlin")

package com.dmall.kt

import kotlin.reflect.KClass

class KotlinMain {

    @JvmField
    var id: Int = 0
    val name: String = "yelong"

}

fun main(args: Array<String>) {


    testClass(JavaMain::class.java)
    testClass(KotlinMain::class)

    println("冲突关键字：${JavaMain.`in`}")
}

fun echo(str: String) {
    println("我是字符串:$str")
}

fun testClass(clazz: Class<JavaMain>) {
    println(clazz.simpleName)
}

fun testClass(clazz: KClass<KotlinMain>) {
    println(clazz.simpleName)
}

class Provider(defaultName: String) {

    private val prefix = "JavaMain "
    var name: String = defaultName
        get() = prefix + field

}

object Singleton {

    lateinit var provider: Provider
    const val VERSION = 7

    fun sayMessage(msg: String) {
        println(msg)
    }

    @JvmStatic
    fun printName(name: String) {
        println("我是名字是：$name")
    }
}
```