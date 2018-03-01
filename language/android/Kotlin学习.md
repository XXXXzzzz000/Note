# Kotlin 学习

阅读地址:<https://www.gitbook.com/book/wangjiegulu/kotlin-for-android-developers-zh/details>
在线kotlin:<https://try.kotlinlang.org/#/Examples/Hello,%20world!/A%20multi-language%20Hello/A%20multi-language%20Hello.kt>

## 类继承

基类:Test1
子类:Test2
**注意**:

1. 父类的open关键字
2. 子类继承的方式

```Kotlin
package com.example.xuanze.testapplication1

open class Test1(name : String)
class Test2(name : String,supername:String):Test1(supername)


fun main(args: Array<String>) {
    var test = Test1("hello")
    var test2 = Test2("111","222")
    print("test:"+test+"test2:"+test2)
//    为什么这段代码无法运行
}
```

## 函数

1. fun关键字
2. 返回值的两种方式(hello,add)
3. 不指定或指定返回值为`Unit`时返回`Unit`类型,与java的`void`类似但是`Unit`是一个对象

```kotlin
package com.example.xuanze.testapplication1

open class Test1(name : String)
{
    var testname=name
    fun hello():Boolean
    {
        print("hello")
        print(testname)
        return false
    }
    fun hello2():Unit{
        print("这个函数返回值为Unit")
    }

    fun add(x:Int,y:Int):Int=x+y

}

fun main(args: Array<String>) {
    var test=Test1("ssss")
    test.hello()
}
```

## 构造方法和函数参数
