---
title: 关于go里面的context 
date: 2018-08-22 09:08:18
tags:
  - Go 
  - golang
categories:
  - 技术
---

##### context的应用场景
经常在go里面遇到context，所以这次特意查看了下context的相关资料，感觉学习就应该要如此，遇到不认识的不懂的，应该要花点时间去弄懂弄通，这样才能慢慢的把自己的知识漏洞给补好。
golang的Context包，是专门用来简化对于处理单个请求的多个goroutine之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个API调用。
<!--more-->
比如有一个网络请求Request，每个Request都需要开启一个goroutine做一些事情，这些goroutine又可能会开启其他的goroutine。这样的话， 我们就可以通过Context，来跟踪这些goroutine，并且通过Context来控制他们的目的，这就是Go语言为我们提供的Context，中文可以称之为“上下文”。在go的http框架gin中就是大量的存在context。
另外一个实际例子是，在Go服务器程序中，每个请求都会有一个goroutine去处理。然而，处理程序往往还需要创建额外的goroutine去访问后端资源，比如数据库、RPC服务等。由于这些goroutine都是在处理同一个请求，所以它们往往需要访问一些共享的资源，比如用户身份信息、认证token、请求截止时间等。而且如果请求超时或者被取消后，所有的goroutine都应该马上退出并且释放相关的资源。这种情况也需要用Context来为我们取消掉所有goroutine。联系到我们之前写的{% post_link go-goroutine %}, 好像我们又多了一种方式，使用context去控制goroutine。

##### context接口
```
type Context interface {

    Deadline() (deadline time.Time, ok bool)

    Done() <-chan struct{}

    Err() error

    Value(key interface{}) interface{}

}
```
在context包里面，我们可以看到Context是一个接口，Context定义了4个方法:
- Deadline方法是获取设置的截止时间的意思，第一个返回式是截止时间，到了这个时间点，Context会自动发起取消请求；第二个返回值ok==false时表示没有设置截止时间，如果需要取消的话，需要调用取消函数进行取消。
- Done方法返回一个只读的chan，类型为struct{}，我们在goroutine中，如果该方法返回的chan可以读取，则意味着parent context已经发起了取消请求，我们通过Done方法收到这个信号后，就应该做清理操作，然后退出goroutine，释放资源。之后，Err 方法会返回一个错误，告知为什么 Context 被取消。
- Err方法返回取消的错误原因，因为什么Context被取消。
- Value方法获取该Context上绑定的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值一般是线程安全的。

##### Context源码解读
在context包里，我们看到了针对Context的实现`emptyCtx`，对于`emptyCtx`，官方的解读是
>  An emptyCtx is never canceled, has no values, and has no deadline. It is not
 struct{}, since vars of this type must have distinct addresses.

另外，在context包里，官方给也给我们提供了2个生成`emptyCtx`的方法：
```
var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

// Background returns a non-nil, empty Context. It is never canceled, has no
// values, and has no deadline. It is typically used by the main function,
// initialization, and tests, and as the top-level Context for incoming
// requests.
func Background() Context {
    return background
}

// TODO returns a non-nil, empty Context. Code should use context.TODO when
// it's unclear which Context to use or it is not yet available (because the
// surrounding function has not yet been extended to accept a Context
// parameter). TODO is recognized by static analysis tools that determine
// whether Contexts are propagated correctly in a program.
func TODO() Context {
    return todo
}

```
`Background()`和`TODO()`方法，他们都返回`emptyCtx`, 不能被取消，不能设置value，不能设置截止时间，所以说，它是作为一个根Context的存在，作为代表性的用于main函数，测试等

那有了以上的根Context，我们发现，源码包里还给我们提供了4个With函数：
```
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)

func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

func WithValue(parent Context, key, val interface{}) Context
```
这些With函数，不紧继承了根的Context，还基于根Context对它进行了拓展，实现了我们一棵树状的Context。这些函数给我们拓展了哪些功能：
- WithCancel函数，传递一个父Context作为参数，返回子Context，以及一个取消函数用来取消Context。 
- WithDeadline函数，和WithCancel差不多，它会多传递一个截止时间参数，意味着到了这个时间点，会自动取消Context，当然我们也可以不等到这个时候，可以提前通过取消函数进行取消。
- WithTimeout和WithDeadline基本上一样，这个表示是超时自动取消，是多少时间后自动取消Context的意思。
- WithValue函数和取消Context无关，它是为了生成一个绑定了一个键值对数据的Context，这个绑定的数据可以通过Context.Value方法访问到

##### 参考文档
https://golang.org/pkg/context/



