#Kotlin标准库函数分析

标准函数有run、apply、let、also、takeIf、takeUnless和with。除了with不是扩展函数，其它都是扩展函数。共同点是都有返回值。而区别就是对原有的值做了那些操作，然后如何返回最终的值。

###run和apply
对象为函数的参数，函数块内用this指代自己，区别就是返回值不一样。run的返回值是最后一行或者指定return；apply的返回值是自己，在Android中适合初始化View的相关属性。

```
fun runMethod() {
    val result = run {
        val message = "Hello"
        "$message Long"
    }

    println("result==$result")

    val name = "Ye".run {
        println("字符串的长度为" + this.length)
        "$this Long"
    }

    println("name==$name")
}

fun applyMethod(){

    val result  = "Hello".apply {
        println("字符串的长度为" + this.length)
        val message = "我是叶龙"
    }

    println("result==$result")
}

```

###let和also
对象为函数的参数，函数块内用it指代自己，区别就是返回值不一样。let的返回值是最后一行或者指定return；also的返回值是自己。

```
fun letMethod() {

    val result = "Hello".let {
        // 用it指代自己
        println("字符串的长度为" + it.length)
        "$it Long"
    }

    println("result==$result")
}

fun alsoMethod(){

    val result = "Hello".also {
        // 用it指代自己
        println("字符串的长度为" + it.length)
    }

    println("result==$result")
}

```

###takeIf、takeUnless
takeIf调用predicate函数进行判断，如果是true返回它本身，否则返回null。takeUnless恰好和takeIf相反

###with
这不是对象的扩展函数，函数块内用this指向传入参数receiver，返回值是最后一行或者指定return

```
fun withMethod(){

    val result  = with("Ye"){
        val message =  "$this Long"
        3
    }

    println("result==$result")
}

```