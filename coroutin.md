```kotlin
fun main(args: Array<String>) {
    loadImage {
        val result = download("asdf")
        log("5获取到结果")
        log("6" + result)
    }
    log("8主线程中 协程方法之后")
}
//执行协程
fun loadImage(block: suspend () -> Unit) {
    log("1开始协程")
    block.startCoroutine(BaseContinuation())
    log("7结束协程")
}
```

Lambda表达式block中如果开启了子线程，那么表达式中zi'xian

#### 异步 -- 耗时操作方法

```kotlin
//这个方法中的内容会全都执行。
suspend fun download(url: String) = suspendCoroutine<String> { continuation ->
    log("2suspend方法 执行异步任务前")
    Thread {//.....耗时操作--异步
        run {
            println("--------------")
            Thread.sleep(1000)
            log("3异步任务正在执行中")
            val s = "异步加载到数据"//获取到的结果
            s.let(continuation::resume)//此时返回的结果仍在子线程中, 需要自己手动切回主线程
           }
        }
    }.start()
	Thread.sleep(200)
    log("4suspend方法 thread run")
}
```

22:37:42:717 [main] 1开始协程
22:37:42:724 [main] 2suspend方法 执行异步任务前

------ --- ------ -----								//说明线程已经启动
22:37:42:725 [main] 4suspend方法 thread run		//休眠后正常执行
22:37:42:725 [main] 7结束协程
22:37:42:725 [main] 8主线程中 协程方法之后
22:37:47:737 [Thread-0] 3异步任务正在执行中
22:37:47:737 [Thread-0] 5获取到结果
22:37:47:737 [Thread-0] 6异步加载到数据

`startCoroutine(BaseContinuation())`执行后
suspend修饰的方法中的内容会全都执行。但是这个方法后面的代码会在该方法返回结果后才会执行，并且默认是在子线程中。 如果拿到结果后要更新UI需要自己切换到主线程。 所以suspend方法会顺利立即执行完，它的子线程会被挂起，与suspend方法在同一个Lambda表达式中suspend方法的代码要等到suspend方法有返回值时再执行。

#### 异步--耗时操作并切回到主线程

```kotlin
suspend fun download(url: String) = suspendCoroutine<String> { continuation ->
    //.....耗时操作--异步
    log("2suspend方法 执行异步任务前")
    Thread {
        run {
            Thread.sleep(1000)
            log("3异步任务正在执行中")
            val s = "异步加载到数据"//获取到的结果
            SwingUtilities.invokeLater {
                //切换到主线程
                continuation.resume(s)
            }
        }
    }.start()
    Thread.sleep(200)
    log("4suspend方法 thread run")
}
```

23:16:06:227 [main] 1开始协程
23:16:06:234 [main] 2suspend方法 执行异步任务前
23:16:06:450 [main] 4suspend方法 thread run
23:16:06:450 [main] 7结束协程
23:16:06:450 [main] 8主线程中 协程方法之后
23:16:07:235 [Thread-0] 3异步任务正在执行中
23:16:07:322 [AWT-EventQueue-0] 5获取到结果
23:16:07:322 [AWT-EventQueue-0] 6异步加载到数据

**continuation.resume(s)是否是在主线程中执行，决定了suspend修饰方法后面的代码执行时所在的线程**





同步 --耗时操作方法

```kotlin
//这个方法中执行耗时操作
suspend fun download(url: String) = suspendCoroutine<String> { continuation ->
    //.....耗时操作
    log("2suspend方法 执行异步任务前")
    Thread.sleep(5000)
    log("3异步任务正在执行中")
    val s = "异步加载到数据"//获取到的结果
    s.let(continuation::resume)//此时返回的结果仍在子线程中, 需要自己手动切回主线程
    log("4suspend方法 thread run")
}
```

没有开启子线程的结果

22:45:21:416 [main] 1开始协程
22:45:21:423 [main] 2suspend方法 执行异步任务前
22:45:26:423 [main] 3异步任务正在执行中
22:45:26:423 [main] 4suspend方法 thread run
22:45:26:424 [main] 5获取到结果
22:45:26:424 [main] 6异步加载到数据
22:45:26:424 [main] 7结束协程
22:45:26:424 [main] 8主线程中 协程方法之后







