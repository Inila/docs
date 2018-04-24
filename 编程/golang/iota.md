# iota: golang中优雅的常量

## 自增长

在 golang 中, 一个方便的习惯就是使用 `iota` 标示符,它简化了常量用于增长数字的定义,给以上相同的值以准确的分类。

```go
const (
    CategoryBooks = iota // 0
    CategoryHealth       // 1
    CategoryClothing     // 2
)
```

## 自定义类型

自增长常量疆场包含一个自定义类型,允许你依靠编译器。

```go
type Steretype int

const (
    TypicalNoob Steretype = iota  // 0
    TypicalHipster                // 1
    TypicalUnixWizard             // 2
    TypicalStartupFounder         // 3
)
```

如果一个函数以 `int` 作为它的参数而不是 `StereoType`, 如果你给他传递一个 `Stereotype`, 它将在编译器期出现问题。

```go
func CountAllTheThings(i int) string {
    return fmt.Sprintf("there are %d things", i)
}

func main() {
    n := TypicalHipster
    fmt.Println(CountAllTheThings(n))
}

// output:
// connot use TypicalHipster (type Stereotype) as type int in argument to CountAllTheThings
```

相反亦是成立的。给一个函数以`Stereotype`作为参数,你不能给它传递 `int`。

```go
func SoSayethThe(character Stereotype) string {
    var s string
    switch character {
    case TypicalNoob:
        s = "I'm a confused ninja rockstar."
    case TypicalHipster:
        s = "Everything was better we programmed uphill and barefoot in the snow on the SUTX 5918"
    case TypicalUnixWizard:
        s = "sudo grep awk sed %!#?!!1!"
    case TypicalStartupFounder:
        s = "exploit compelling convergence to syndicate geo-targeted solutions"
    }
    return s
}

func main() {
    i := 2
    fmt.Println(SoSayethThe(i))
}

// output:
// cannot use i (type int) as type Stereotype in argument to SoSayethThe
```

这是一个戏剧性的转折,尽管如此。你可以传递一个数值常量,然后他能工作。

```go
func main() {
    fmt.Println(SoSayethThe(0))
}

// output:
// I'm a confused ninja rockstar.
```
这是因为常量在Go中是弱类型直到它使用在一个严格的上下文环境中。

## Skipping Values

设想你在处理消费者的音频输出。音频可能无论什么都没有任何输出，或者它可能是单声道，立体声，或是环绕立体声的。
这可能有些潜在的逻辑定义没有任何输出为0,单声道为1,立体声为2,值是由通道的数量提供。
所以你给 Dolby 5.1 环绕立体声什么值。
一方面，它有6个通道输出，但是另一方面，仅仅 5 个通道是全带宽通道（因此 5.1 称号 - 其中 .1 表示的是低频效果通道）。
不管怎样，我们不想简单的增加到 3。
我们可以使用下划线跳过不想要的值。

```go
type AudioOutput int

const (
    OutMute AudioOutput = iota // 0
    OutMono                    // 1
    OutStereo                  // 2
    _
    _
    OutSurround                // 5
)
```

## 表达式

`iota` 可以做更多事情,而不仅仅是 increment。更精确说, `iota`总是用于 increment,但是它可以用于表达式,在常量中存储结果值。

这是我们创建一个常量用于位掩码。

```go
type Allergen int

const (
    IgEggs Allergen = 1 << iota // 1 << 0 which is 00000001
    IgChocolate                         // 1 << 1 which is 00000010
    IgNuts                              // 1 << 2 which is 00000100
    IgStrawberries                      // 1 << 3 which is 00001000
    IgShellfish                         // 1 << 4 which is 00010000
)
```

这个工作是因为当你在一个 `const` 组中仅仅有一个标识符在一行的时候,它将使用增长的 `iota` 取得前面的表达式并且再运用它。
在Go语言的 spec 中, 这就是所谓的隐形重复最后一个非空表达式列表。

如果你对鸡蛋，巧克力和海鲜过敏，把这些 bits 翻转到 “on” 的位置（从左到右映射 bits）。然后你将得到一个 bit 值 00010011，它对应十进制的 19。

```go
fmt.Println(IgEggs | IgChocolate | IgShellfish)

// output:
// 19
```

这是在 [Effective Go](https://golang.org/doc/effective_go.html#constants) 中一个非常好定义数量及的示例:

```go
type ByteSize float64

const (
    _               = iota                  // ignore first value by assingning to blank identifier
    KB ByteSize     = 1 << (10 * iota)      // 1 << (10*1)
    MB                                      // 1 << (10*2)
    GB                                      // 1 << (10*3)
    TB                                      // 1 << (10*4)
    PB                                      // 1 << (10*5)
    EB                                      // 1 << (10*6)
    ZB                                      // 1 << (10*7)
    YB                                      // 1 << (10*8)
)
```

## 还有更多

当你在把两个常量定义在一行的时候会发生什么？
Banana的值是什么?还是3？Durian的值又是？

```go
const (
    Apple, Banana = iota + 1, iota + 2
    Cherimoya, Durian
    Elderberry, Fig
)
```

`iota` 在下一行增长, 而不是立即取得他的引用。

```go
// Apple: 1
// Banana: 2
// Cherimoya: 2
// Durian: 3
// Elderberry: 3
// Fig: 4
```
这搞砸了，因为现在你的常量有相同的值。

在 Go 中, 关于常量有很多东西可以说, 你应该在 golang 博客读取 Rob Pike 的 [这篇博客](https://blog.golang.org/constants)