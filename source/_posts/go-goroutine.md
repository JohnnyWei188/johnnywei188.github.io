---
title: golang控制goroutine
tags:
  - Go 
  - golang
categories:
  - 技术
---
golang中的goroutine启动之后，因为不像其他的语言启动线程之后有个句柄可以对线程进行关闭或者其他的操作，所以很不好控制，最近就遇到需要gotoutine关闭重新加载配置的这一项操作。记录下goroutine重启的方案。

##### 如何把当前已经启动的goroutine退出，退出后再另外拉起新的goroutine
需要说明的是，我这个应用是一个服务型的应用，goroutine拉起之后以一个常驻的线程在服务端, 代码如下：
```
package main

import (
    "fmt"
    "time"
)

var (
    control, close = make(chan int, 1), make(chan int, 1)
    config         = map[string]string{
        "a": "a",
    }
)

func main() {
    fmt.Println("start...")
    dosomething(config)
    <-close
}

func dosomething(config map[string]string) {
    go gorou(config)
}

func gorou(config map[string]string) {
    fmt.Println("config...", config)
    i := 0
    for {
        select {
        case <-control:
            fmt.Println("restart...")
            return
        default:
            time.Sleep(1 * time.Second)
            i++
            fmt.Println(i)
            if i >= 3 {
                control <- 1
                //update config or do other things
                args := map[string]string{
                    "restart": "restart...map",
                    "time":    time.Now().String(),
                }
                dosomething(args)
            }
        }
    }
}
```

##### 总结
在实现这个功能的时候有遇到问题，就是在初始化control的时候`control = make(chan int)`, 这样初始化其实没有问题的，但是在执行到control<-1却挂了，提示：
```
start...
config... map[a:a]
1
2
3
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
        /Users/Johnny/go/src/github.com/JohnnyWei188/learn/goroutine_exit3.go:18 +0x94

goroutine 5 [chan send]:
main.gorou(0xc420072180)
        /Users/Johnny/go/src/github.com/JohnnyWei188/learn/goroutine_exit3.go:38 +0x18d
created by main.dosomething
        /Users/Johnny/go/src/github.com/JohnnyWei188/learn/goroutine_exit3.go:22 +0x3f
exit status 2
```
原因是因为，初始化control如果没有指定chan的capacity，那么control就是一个无缓冲的阻塞的channel，那么在执行到i=3的时候，写入一个无缓冲的channe后，然后阻塞在这当前这个线程形成死锁，所以，线程在等一个永远不会来的数据，那整个程序就永远等下去了。 这显然是没有结果的，所以主程序就自己杀掉自己，报告出来。

在我们把`control=make(chan int, 1)`之后，问题得到解决，这是因为我们在`control<-1`写入数据的时候，channel因为是有buffer缓冲的，不会操作程序的阻塞，说程序继续往下执行后，在dosomething里面由重新拉起了goroutine，使我们的程序得以继续。那如果后面的代码不能再拉起一个goroutine，那程序是不是一样又会进入死锁呢？

我们可以把
```
//args := map[string]string{
//    "restart": "restart...map",
//    "time":    time.Now().String(),
//}
//dosomething(args)
```
这几行代码注释掉进行验证，注释掉后我们在执行程序，发现果然报错了，
```
start...
config... map[a:a]
1
2
3
restart...
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
        /Users/Johnny/go/src/github.com/JohnnyWei188/learn/goroutine_exit3.go:18 +0x94
exit status 2
```
原因是因为，在return之后，我们的goroutinue就结束了，那main的goroutine认为自己也是在等一个没有结果的channel，所以也主动把自己给close掉了。

