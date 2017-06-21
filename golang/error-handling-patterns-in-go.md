# Go语言错误处理模型

May 9, 2017

Go语言一个主要的优点是其错误模型。可是我并不喜欢它 － 它有时候像[这样](http://joeduffyblog.com/2016/02/07/the-error-model/) － 不过它还是最好的方式之一。已经有很多关于Go里错误处理的最佳实践文章了([错误处理和Go](https://blog.golang.org/error-handling-and-go)与[不要只是检查，要优雅地处理错误](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)都是很好的资源)。不过本文会讨论一些虽然鲜为人知，但对于编写简单可靠的Go代码来说，仍然很重要的错误处理模式。

## 总为错误做检查

几乎所有人都认同在Go里应该总是检查错误(像[errcheck](https://github.com/kisielk/errcheck)这样的工具能帮你做)。而且同样适用于用defer语句释放资源的场景。这两者结合后出现了一个更有意思的场景 － 怎么处理被defer语句调用的函数所返回的错误？经常能看到的是这样的：

```
resp, err := http.Get(url)

if err != nil {
    return err
}

defer resp.Body.Close()
```

尽管调用Close可能会返回一个错误，但是我们还是可以把它忽略掉，这一般不会牺牲正确性。但这并不是说我们就总能忽略被延迟调用的函数所返回的错误了。思考下面这个向文件里写数据的函数(暂时先忽略错误，ioutil.WriteFile函数里几乎做了同样的事)：

```
func WriteFile(filename string, data []byte) error {
    f, err := os.Create(filename)

    if err != nil {
        return err
    }

    defer f.Close()

    _, err = f.Write(data)
    return err
}
```

上面的代码表面上看起来不错，但其实有个严重的问题。Linux系统调用的手册上说：

>不检查close()的返回值是一个常见的，但仍然严重的编程错误。很有可能前一次write(2)操作的错误在最后的close()才被第一次爆出来。关闭文件时不检查返回值可能会导致静默的数据丢失。

现在知道了调用Close时也要检查错误，那最好的检查方式是啥？下面可能是一种解决方案：

```
func WriteFile(filename string, data []byte) (err error) {
    f, err := os.Create(filename)

    if err != nil {
        return err
    }

    defer func() {
        if cerr := f.Close(); cerr != nil && err == nil {
            err = cerr
        }
    }()

    _, err = f.Write(data)
    return err
}
```

这版代码还是在defer语句里调用Close(这里并不严格需要defer语句，因为if后面只有一个Write － 如果不用defer就可以把上面的代码重新排序下，会更简单点 － 但一般来说使用defer的方式会更安全)，但这次我们正确的检查了错误，并且在Close调用产生错误时更新了函数的返回值。这里修改了函数的签名，用有命名的返回值来实现(如果你对这种方式不熟悉可以看看这篇好[文章](https://blog.golang.org/defer-panic-and-recover))。

上面这种方式是正确的，但把代码变丑了：要写五行代码去处理和这个函数的主要任务无关的情况，而且还必须要跟踪两种不同的错误，不是一种！如果有多个资源要以这种方式释放，代码就会变得更丑。我通常会定义帮助函数safeClose来处理这种问题：

```
func safeClose(c io.Closer, err *error) {
    if cerr := c.Close(); cerr != nil && *err == nil {
        *err = cerr
    }
}

func WriteFile(filename string, data []byte) (err error) {
    f, err := os.Create(filename)

    if err != nil {
        return err
    }

    defer safeClose(f, &err)

    _, err = f.Write(data)
    return err
}
```

这段代码和第一个例子看起来几乎一样，但这次真的正确了。我曾考虑搞一个public包暴露这个函数，但还是决定不搞了，因为这个函数不是一个足够好的public抽象。

如果你不确信上面这种做法都是必要的，请看看标准库ioutil.WriteFile的实现：

```
func WriteFile(filename string, data []byte, perm os.FileMode) error {
    f, err := os.OpenFile(filename, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, perm)
    if err != nil {
        return err
    }
    n, err := f.Write(data)
    if err == nil && n < len(data) {
        err = io.ErrShortWrite
    }
    if err1 := f.Close(); err == nil {
        err = err1
    }
    return err
}
```

它没有用defer，但其实和上面的做法是一样的。所以即使write没有返回错误，那也不意味写操作就成功了：必须还得检查了Close的返回值后才能确认。

## 不要不动脑子就用 err != nil

希望咱们都同意应该检查每一个错误。但这并不是说要把`if err != nil { ... }`放到每个需要检查的地方：它把代码搞的不可读了，弄的更难定位真正的bug了。思考下面这个例子(有删节)，是我写的一段真正在用的代码(原本的结构体有更多的字段)：

```
type point struct {
    Longitude     float32
    Latitude      float32
    Distance      int32
    ElevationGain int16
    ElevationLoss int16
}
```

这个结构体有几个不同类型的字段，服务端用Big Endian格式把它们编码后发给客户端。如果要从输入流中把这些字段解析到这个结构体的指针上，可能刚开始会写出这样的代码：

```
func parse(r io.Reader) (*point, error) {
    var p point

    if err := binary.Read(r, binary.BigEndian, &p.Longitude); err != nil {
        return nil, err
    }

    if err := binary.Read(r, binary.BigEndian, &p.Latitude); err != nil {
        return nil, err
    }

    if err := binary.Read(r, binary.BigEndian, &p.Distance); err != nil {
        return nil, err
    }

    if err := binary.Read(r, binary.BigEndian, &p.ElevationGain); err != nil {
        return nil, err
    }

    if err := binary.Read(r, binary.BigEndian, &p.ElevationLoss); err != nil {
        return nil, err
    }

    return &p, nil
}
```

这代码太烂了 － 简直看一眼我就头疼。下面这种一元错误处理方式可以补救下：只需要定义一个帮助结构体，它仅持有一个真实的io.Reader，一个错误，以及一个read函数，这个函数只在前一次调用底层Read没出错的情况下，才继续下次的Read调用。这样在所有的读操作完成后，只需要做一次错误检查就行了。下面是一个更完善的例子：

```
type reader struct {
    r   io.Reader
    err error
}

func (r *reader) read(data interface{}) {
    if r.err == nil {
        r.err = binary.Read(r.r, binary.BigEndian, data)
    }
}

func parse(input io.Reader) (*point, error) {
    var p point
    r := reader{r: input}

    r.read(&p.Longitude)
    r.read(&p.Latitude)
    r.read(&p.Distance)
    r.read(&p.ElevationGain)
    r.read(&p.ElevationLoss)

    if r.err != nil {
        return nil, r.err
    }

    return &p, nil
}
```

这就好多了，主线代码可读了，一眼就能看出来代码干了啥。但这块技术不好的地方是没法通用化 － 因为必须要为每个函数都搞个包装类型。但好事是对io.Reader和io.Writer来说都适合，也包括很多其它标准库里的读/写函数。

如果你的代码正在做大量的错误处理，可以考虑用这种方式把函数精简的更易读。

## 给错误添加上下文

我在准备这篇文章的草稿时，Coda Hale发[推特](https://twitter.com/coda/status/859848060058296320)说了比我自己在这部分上能写的都更好的介绍：

>“Go的错误机制棒极了，因为必须要处理它们，但应该把错误封装起来，压栈跟踪信息，这就更好了。”我也喜欢检查异常。

下面这种仅仅把错误传递给调用者的做法挺诱人的：

```
if err != nil {
    return err
}
```

但是也许你并不想在高层函数里看到低层的错误：高层的调用者在面对深层调用栈里调用strconv.Atoi返回的“invalid syntax”错误时能做啥？这也就是为啥要鼓励Go程序员要么处理错误，要么把错误和一些额外的信息封装起来的原因：

```
if err != nil {
    return fmt.Errorf("something failed: %v", err)
}
```

这样就好多了，对大多数场景来说都不错，但我并不喜欢这种方式。难道2017年最好的错误封装方式就是用printf风格的函数吗？还好Dave Cheney写了更好的[errors](https://github.com/pkg/errors)包，它提供了更好的把上下文添加到错误里的方式：

```
if err != nil {
    return errors.Wrap(err, "something failed")
}
```

它还能解封装错误，并压栈跟踪信息到错误里。用了它就不需要标准库里的errors和fmt了，因为它还有New和Errorf函数。

我特喜欢这个包，新项目里拉取的第一个依赖经常就是它(我打算放到其它时候再讨论是否值得为一些如此简单的东西依赖其它包)。依我看，这个包应该首先成为标准库的一部分。

[原文链接](https://mijailovic.net/2017/05/09/error-handling-patterns-in-go/)