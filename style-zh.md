<!--

Editing this document:

- Discuss all changes in GitHub issues first.
- Update the table of contents as new sections are added or removed.
- Use tables for side-by-side code samples. See below.

Code Samples:

Use 2 spaces to indent. Horizontal real estate is important in side-by-side
samples.

For side-by-side code samples, use the following snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
BAD CODE GOES HERE
```

</td><td>

```go
GOOD CODE GOES HERE
```

</td></tr>
</tbody></table>
~~~

(You need the empty lines between the <td> and code samples for it to be
treated as Markdown.)

If you need to add labels or descriptions below the code samples, add another
row before the </tbody></table> line.

~~~
<tr>
<td>DESCRIBE BAD CODE</td>
<td>DESCRIBE GOOD CODE</td>
</tr>
~~~

-->

# Uber Go Style Guide
# Uber Go 代码风格指南

## Table of Contents
## 目录

- [Uber Go Style Guide](#uber-go-style-guide)
- [Uber Go 代码风格指南](#uber-go-代码风格指南)
  - [Table of Contents](#table-of-contents)
  - [目录](#目录)
  - [Introduction](#introduction)
  - [简介](#简介)
  - [Guidelines](#guidelines)
  - [指南](#指南)
    - [Pointers to Interfaces](#pointers-to-interfaces)
    - [指向接口的指针](#指向接口的指针)
    - [Receivers and Interfaces](#receivers-and-interfaces)
    - [接收器和接口](#接收器和接口)
    - [Zero-value Mutexes are Valid](#zero-value-mutexes-are-valid)
    - [Copy Slices and Maps at Boundaries](#copy-slices-and-maps-at-boundaries)
    - [复制切片(Slice)和映射(Map)时，注意边界](#复制切片slice和映射map时注意边界)
      - [Receiving Slices and Maps](#receiving-slices-and-maps)
      - [复制切片(Slice)和映射(Map)](#复制切片slice和映射map)
      - [Returning Slices and Maps](#returning-slices-and-maps)
      - [返回切片(Slice)和映射(Map)](#返回切片slice和映射map)
    - [Defer to Clean Up](#defer-to-clean-up)
    - [延迟清除](#延迟清除)
    - [Channel Size is One or None](#channel-size-is-one-or-none)
    - [Channel 有或无](#channel-有或无)
    - [Start Enums at One](#start-enums-at-one)
    - [枚举从一开始](#枚举从一开始)
    - [Error Types](#error-types)
    - [错误类型](#错误类型)
    - [Error Wrapping](#error-wrapping)
    - [封装错误](#封装错误)
    - [Handle Type Assertion Failures](#handle-type-assertion-failures)
    - [处理类型断言失败](#处理类型断言失败)
    - [Don't Panic](#dont-panic)
    - [不要 panic](#不要-panic)
    - [Use go.uber.org/atomic](#use-gouberorgatomic)
    - [使用 go.uber.org/atomic](#使用-gouberorgatomic)
  - [Performance](#performance)
  - [性能](#性能)
    - [Prefer strconv over fmt](#prefer-strconv-over-fmt)
    - [优先使用 strconv 而不是 fmt](#优先使用-strconv-而不是-fmt)
    - [Avoid string-to-byte conversion](#avoid-string-to-byte-conversion)
    - [避免 string 到 byte 的转换](#避免-string-到-byte-的转换)
  - [Style](#style)
  - [代码风格](#代码风格)
    - [Group Similar Declarations](#group-similar-declarations)
    - [相似的声明放在一组](#相似的声明放在一组)
    - [Import Group Ordering](#import-group-ordering)
    - [import组内的包导入顺序](#import组内的包导入顺序)
    - [Package Names](#package-names)
    - [包名](#包名)
    - [Function Names](#function-names)
    - [函数名称](#函数名称)
    - [Import Aliasing](#import-aliasing)
    - [导入别名](#导入别名)
    - [Function Grouping and Ordering](#function-grouping-and-ordering)
    - [函数分组与排序](#函数分组与排序)
    - [Reduce Nesting](#reduce-nesting)
    - [减少嵌套](#减少嵌套)
    - [Unnecessary Else](#unnecessary-else)
    - [不必要的 Else](#不必要的-else)
    - [Top-level Variable Declarations](#top-level-variable-declarations)
    - [顶级变量声明](#顶级变量声明)
    - [Prefix Unexported Globals with _](#prefix-unexported-globals-with-_)
    - [对于未导出的全局变量，使用_作为前缀](#对于未导出的全局变量使用_作为前缀)
    - [Embedding in Structs](#embedding-in-structs)
    - [结构体的嵌入](#结构体的嵌入)
    - [Use Field Names to initialize Structs](#use-field-names-to-initialize-structs)
    - [使用字段名来初始化结构体](#使用字段名来初始化结构体)
    - [Local Variable Declarations](#local-variable-declarations)
    - [本地变量声明](#本地变量声明)
    - [nil is a valid slice](#nil-is-a-valid-slice)
    - [nil 是一个有效的分片](#nil-是一个有效的分片)
    - [Reduce Scope of Variables](#reduce-scope-of-variables)
    - [缩小变量作用域](#缩小变量作用域)
    - [Avoid Naked Parameters](#avoid-naked-parameters)
    - [避免语意不明的参数](#避免语意不明的参数)
    - [Use Raw String Literals to Avoid Escaping](#use-raw-string-literals-to-avoid-escaping)
    - [使用Raw String Literals，避免转义](#使用raw-string-literals避免转义)
    - [Initializing Struct References](#initializing-struct-references)
    - [初始化结构引用](#初始化结构引用)
    - [Format Strings outside Printf](#format-strings-outside-printf)
    - [在 Printf 之外格式化字符串](#在-printf-之外格式化字符串)
    - [Naming Printf-style Functions](#naming-printf-style-functions)
    - [命名 Printf 样式的函数](#命名-printf-样式的函数)
  - [Patterns](#patterns)
  - [编程模式](#编程模式)
    - [Test Tables](#test-tables)
    - [Test 表](#test-表)
    - [Functional Options](#functional-options)
    - [功能选项](#功能选项)

## Introduction
## 简介

Styles are the conventions that govern our code. The term style is a bit of a
misnomer, since these conventions cover far more than just source file
formatting—gofmt handles that for us.

代码风格是管理代码的一些约定。代码风格这个术语有点不是特别恰当，因为这些约定所涉及的范围远远不止源文件格式，源文件格式可以由 `gofmt` 为我们处理。

The goal of this guide is to manage this complexity by describing in detail the
Dos and Don'ts of writing Go code at Uber. These rules exist to keep the code
base manageable while still allowing engineers to use Go language features
productively.

本指南的目的是通过详细描述在 Uber 编写 Go 代码的注意事项来管理代码复杂性。这些规则的存在是为了使代码库易于管理，同时仍然允许工程师有效地使用 Go 语言特性。

This guide was originally created by [Prashant Varanasi] and [Simon Newton] as
a way to bring some colleagues up to speed with using Go. Over the years it has
been amended based on feedback from others.

本指南最初是由 [Prashant Varanasi] 和 [Simon Newton] 创建，目的是为了让一些同事可以快速上手 Go 语言。多年以来，已经根据其他人的反馈做了改进。

  [Prashant Varanasi]: https://github.com/prashantv
  [Simon Newton]: https://github.com/nomis52

This documents idiomatic conventions in Go code that we follow at Uber. A lot
of these are general guidelines for Go, while others extend upon external
resources:

本文档记录了我们在 Uber 遵循的 Go 代码中的惯用约定。 其中许多是 Go 的通用准则，而其他准则基于外部资源扩展来的：

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [The Go common mistakes guide](https://github.com/golang/go/wiki/CodeReviewComments)

All code should be error-free when run through `golint` and `go vet`. We
recommend setting up your editor to:

通过 `golint` 和 `go vet` 运行时，所有代码均应无错误。建议将编辑器设置为：

- Run `goimports` on save
- 保存时运行 `goimports`
- Run `golint` and `go vet` to check for errors
- 运行 `golint` and `go vet` 以检查是否含有错误

You can find information in editor support for Go tools here:
<https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins>

可以在这里找到编辑器对 Go 工具支持的信息:
<https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins>

## Guidelines
## 指南

### Pointers to Interfaces
### 指向接口的指针

You almost never need a pointer to an interface. You should be passing
interfaces as values—the underlying data can still be a pointer.

几乎几乎不需要指向接口的指针。您应该将接口作为值传递，基础数据仍然可以是指针。

An interface is two fields:
一个接口有两个字段:

1. A pointer to some type-specific information. You can think of this as
  "type."
1. 一个指向某些特性类型信息的指针。可以将其视为“类型”。
2. Data pointer. If the data stored is a pointer, it’s stored directly. If
  the data stored is a value, then a pointer to the value is stored.
2. 数据指针。如果存储的数据是指针，则直接存储。如果存储的数据是一个值，则存储指向该值的指针。

If you want interface methods to modify the underlying data, you must use a
pointer.

如果想要接口方法修改基础数据，则必须使用指针。

### Receivers and Interfaces
### 接收器和接口

Methods with value receivers can be called on pointers as well as values.

具有值接收器的方法可以在指针和值上调用。

For example,

例如

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// You can only call Read using a value
// 只能用值调用 Read
sVals[1].Read()

// This will not compile:
// 这个将不能编译:
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// You can call both Read and Write using a pointer
// 可以通过指针调用 Read 和 Write
sPtrs[1].Read()
sPtrs[1].Write("test")
```

Similarly, an interface can be satisfied by a pointer, even if the method has a
value receiver.

同样，即使方法具有值接收器，也可以通过指针来满足接口。

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// The following doesn't compile, since s2Val is a value, and there is no value receiver for f.
// 下面的代码不能编译，因为 s2Val 是一个值，而这里没有 f 可用的值接收器。
//   i = s2Val
```

Effective Go has a good write up on [Pointers vs. Values].

Effective Go 有一端关于 [Pointers vs. Values] 很好的说明。

  [Pointers vs. Values]: https://golang.org/doc/effective_go.html#pointers_vs_values

### Zero-value Mutexes are Valid

The zero-value of `sync.Mutex` and `sync.RWMutex` is valid, so you almost
never need a pointer to a mutex.

`sync.Mutex` 和 `sync.RWMutex` 的空值是有效的，所以几乎不需要指向互斥量的指针。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

If you use a struct by pointer, then the mutex can be a non-pointer field or,
preferably, embedded directly into the struct.

如果通过指针使用结构，则互斥体可以是非指针字段，或者最好直接嵌入到结构中。

<table>
<tbody>
<tr><td>

```go
type smap struct {
  sync.Mutex

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

</tr>
<tr>
<td>Embed for private types or types that need to implement the Mutex interface.</td>
<td>For exported types, use a private lock.</td>
</tr>
<tr>
<td>嵌入私有类型或类型实现 Mutex 接口</td>
<td>对于导出类型，请使用专用锁。</td>
</tr>
</tbody></table>

### Copy Slices and Maps at Boundaries
### 复制切片(Slice)和映射(Map)时，注意边界

Slices and maps contain pointers to the underlying data so be wary of scenarios
when they need to be copied.

切片和映射包含指向基础数据的指针，因此在需要复制它们时要特别注意方案。

#### Receiving Slices and Maps
#### 复制切片(Slice)和映射(Map)

Keep in mind that users can modify a map or slice you received as an argument
if you store a reference to it.

请记住，如果持有对切片和映射的引用，则用户可以对其进行修改。

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
// 是要修改 d1.trips 吗？
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// We can now modify trips[0] without affecting d1.trips.
// 现在可以修改 trips[0] 而不会 d1.trips。
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Returning Slices and Maps
#### 返回切片(Slice)和映射(Map)

Similarly, be wary of user modifications to maps or slices exposing internal
state.

同样，请注意用户对暴露内部状态的切片和映射的修改。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  sync.Mutex

  counters map[string]int
}

// Snapshot returns the current stats.
// Snapshot 返回当前状态。
func (s *Stats) Snapshot() map[string]int {
  s.Lock()
  defer s.Unlock()

  return s.counters
}

// snapshot is no longer protected by the lock!
// snapshot 没有被锁保护
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  sync.Mutex

  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.Lock()
  defer s.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
// 现在 Snapshot 是一个副本。
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Defer to Clean Up
### 延迟清除

Use defer to clean up resources such as files and locks.

使用 defer 清理资源，例如文件和锁。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// easy to miss unlocks due to multiple returns
// 由于多次返回，容易错失解锁
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
// 更好的可读性
```

</td></tr>
</tbody></table>

Defer has an extremely small overhead and should be avoided only if you can
prove that your function execution time is in the order of nanoseconds. The
readability win of using defers is worth the miniscule cost of using them. This
is especially true for larger methods that have more than simple memory
accesses, where the other computations are more significant than the `defer`.

Defer的开销非常小，只有在可以证明函数执行时间处于纳秒级的程度时，才应避免使用。 defer 胜在可读性，因为使用成本微不足道。This is especially true for larger methods that have more than simple memory accesses, where the other computations are more significant than the `defer`.

### Channel Size is One or None
### Channel 有或无

Channels should usually have a size of one or be unbuffered. By default,
channels are unbuffered and have a size of zero. Any other size
must be subject to a high level of scrutiny. Consider how the size is
determined, what prevents the channel from filling up under load and blocking
writers, and what happens when this occurs.

通道(Channel)通常有一个大小为一或没有的缓冲(buffered)。默认情况下，通道是没有缓冲或者有一个大小为零的缓冲。任何其他尺寸都必须经过严格的审查。考虑如何确定大小，什么阻止通道在负载下填满并阻止写入器，以及发生这种情况时会发生什么。


<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// Ought to be enough for anybody!
// 对于任何人来说，这些应该够用了。
c := make(chan int, 64)
```

</td><td>

```go
// Size of one
c := make(chan int, 1) // or
// Unbuffered channel, size of zero
c := make(chan int)
```

</td></tr>
</tbody></table>

### Start Enums at One
### 枚举从一开始

The standard way of introducing enumerations in Go is to declare a custom type
and a `const` group with `iota`. Since variables have a 0 default value, you
should usually start your enums on a non-zero value.

在 Go 中引入枚举的标准方法是声明一个自定义类型和一个带有 iota 的 const 组。由于变量的默认值为 0，因此通常应以非零值开头枚举。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

There are cases where using the zero value makes sense, for example when the
zero value case is the desirable default behavior.

在某些情况下，使用零值是有意义的，例如，当零值是理想的默认行为时。

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->

### Error Types
### 错误类型

There are various options for declaring errors:
几种声明错误的可选性：

- [`errors.New`] for errors with simple static strings
- [`errors.New`] 用于简单的错误信息
- [`fmt.Errorf`] for formatted error strings
- [`fmt.Errorf`] 用于格式化的错误信息。
- Custom types that implement an `Error()` method
- 实现 `Error()` 方法的自定义类型
- Wrapped errors using [`"pkg/errors".Wrap`]
- 用 [`"pkg/errors".Wrap`] 包装的错误

When returning errors, consider the following to determine the best choice:

返回错误时，请考虑以下因素以确定最佳选择:

- Is this a simple error that needs no extra information? If so, [`errors.New`]
  should suffice.
- 这是一个不需要额外信息的简单错误吗？如果是， [`errors.New`] 就够用。
- Do the clients need to detect and handle this error? If so, you should use a
  custom type, and implement the `Error()` method.
- 客户需要检测并处理错误吗？如果是这样，则应该使用自定义类型，并实现 `Error()` 方法。
- Are you propagating an error returned by a downstream function? If so, check
  the [section on error wrapping](#error-wrapping).
- 是否正在传播下游函数返回的错误？ 如果是这样，请检查 [section on error wrapping](#error-wrapping)。
- Otherwise, [`fmt.Errorf`] is okay.
- 其余， [`fmt.Errorf`] 就可以。

  [`errors.New`]: https://golang.org/pkg/errors/#New
  [`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf
  [`"pkg/errors".Wrap`]: https://godoc.org/github.com/pkg/errors#Wrap

If the client needs to detect the error, and you have created a simple error
using [`errors.New`], use a var for the error.

如果客户端需要检测到错误，并且已经使用 [`errors.New`] 创建了一个简单的错误，请为该错误使用一个变量。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

If you have an error that clients may need to detect, and you would like to add
more information to it (e.g., it is not a static string), then you should use a
custom type.

如果您有可能需要客户端检测的错误，并且想向其添加更多信息（例如，它不是静态字符串），则应该使用自定义类型。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

Be careful with exporting custom error types directly since they become part of
the public API of the package. It is preferable to expose matcher functions to
check the error instead.

直接导出自定义错误类型时要小心，因为它们将成为程序包公共API的一部分。最好公开匹配功能以检查错误。

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

<!-- TODO: Exposing the information to callers with accessor functions. -->

### Error Wrapping
### 封装错误

There are three main options for propagating errors if a call fails:
调用失败时，有三种主要的错误传播方式：

- Return the original error if there is no additional context to add and you
  want to maintain the original error type.
- 如果没有额外的上下文要添加并且想要维护原始错误类型，则返回原始错误。
- Add context using [`"pkg/errors".Wrap`] so that the error message provides
  more context and [`"pkg/errors".Cause`] can be used to extract the original
  error.
- 使用 [`"pkg/errors".Wrap`] 添加上下文，以便错误消息提供更多上下文，并且 [`"pkg/errors".Cause`] 可用于提取原始错误。
- Use [`fmt.Errorf`] if the callers do not need to detect or handle that
  specific error case.
- 如果调用者不需要检测或处理特定的错误情况，则使用  [`fmt.Errorf`] 。

It is recommended to add context where possible so that instead of a vague
error such as "connection refused", you get more useful errors such as
"call service foo: connection refused".

建议在可能的情况下添加上下文，以使调用者获得诸如“call service foo: connection refused”之类的更有用的错误信息，而不是诸如“connection refused”之类的模糊错误信息。

When adding context to returned errors, keep the context succinct by avoiding
phrases like "failed to", which state the obvious and pile up as the error
percolates up through the stack:

在为返回的错误添加上下文时，请避免使用诸如 “failed to” 之类的短语以保持上下文简洁。这些短语是显而易见的，并在错误穿过堆栈时逐渐堆积：

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```

<tr><td>

```
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```
x: y: new store: the error
```

</td></tr>
</tbody></table>

However once the error is sent to another system, it should be clear the
message is an error (e.g. an `err` tag or "Failed" prefix in logs).

另外，如果将错误发送到另一个系统，应该清楚该消息是一个错误（例如，日志中的 `err` 标签或 “Failed” 前缀）。

参考 [Don't just check errors, handle them gracefully].

  [`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
  [Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

### Handle Type Assertion Failures
### 处理类型断言失败

The single return value form of a [type assertion] will panic on an incorrect
type. Therefore, always use the "comma ok" idiom.

单个形式的 [type assertion] 返回值将会导致对不正确的类型。 因此，请始终使用 “comma ok” 的习惯用法。

  [type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

### Don't Panic
### 不要 panic

Code running in production must avoid panics. Panics are a major source of
[cascading failures]. If an error occurs, the function must return an error and
allow the caller to decide how to handle it.

在生产中运行的代码必须避免使用 panics。 panics 是 [cascading failures] 的主要根源。如果发生错误，该函数必须返回错误，并允许调用方决定如何处理它。

  [cascading failures]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover is not an error handling strategy. A program must panic only when
something irrecoverable happens such as a nil dereference. An exception to this is
program initialization: bad things at program startup that should abort the
program may cause panic.

Panic/recover 不是错误处理策略。仅当发生不可恢复的事情（例如 nil 引用）时，程序才可以 panic。程序初始化是一个例外：在程序启动时，如果有不良情况发生，应中止程序避免 panic。

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Even in tests, prefer `t.Fatal` or `t.FailNow` over panics to ensure that the
test is marked as failed.

即使在测试代码中，也优先使用 `t.Fatal` 或者 `t.FailNow` 而不是 panic 来确保测试被标记为失败。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

<!-- TODO: Explain how to use _test packages. -->

### Use go.uber.org/atomic
### 使用 go.uber.org/atomic

Atomic operations with the [sync/atomic] package operate on the raw types
(`int32`, `int64`, etc.) so it is easy to forget to use the atomic operation to
read or modify the variables.

带有 [sync/atomic] 包的原子操作对原始类型（int32，int64等）进行操作，很容易忘记使用原子操作来读取或修改变量。

[go.uber.org/atomic] adds type safety to these operations by hiding the
underlying type. Additionally, it includes a convenient `atomic.Bool` type.

[go.uber.org/atomic] 通过隐藏基础类型为这些操作增加了类型安全性。另外，它包含一个方便的 `atomic.Bool` 类型。

  [go.uber.org/atomic]: https://godoc.org/go.uber.org/atomic
  [sync/atomic]: https://golang.org/pkg/sync/atomic/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

## Performance
## 性能

Performance-specific guidelines apply only to the hot path.

性能方面的准则仅适用于焦点路径。

### Prefer strconv over fmt
### 优先使用 strconv 而不是 fmt


When converting primitives to/from strings, `strconv` is faster than
`fmt`.

将原语与字符串进行相互转换时，`strconv` 的速度比 `fmt`。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Avoid string-to-byte conversion
### 避免 string 到 byte 的转换

Do not create byte slices from a fixed string repeatedly. Instead, perform the
conversion once and capture the result.

不要重复从固定字符串创建字节片。相反，请执行一次转换并保存结果。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

## Style
## 代码风格

### Group Similar Declarations
### 相似的声明放在一组

Go supports grouping similar declarations.

Go语言支持将相似的声明放在一组。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

This also applies to constants, variables, and type declarations.

这同样适用于常量，变量和类型声明。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

Only group related declarations. Do not group declarations that are unrelated.

仅将相关的放在一组；而不要分组不相关的。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

Groups are not limited in where they can be used. For example, you can use them
inside of functions.

分组声明不受使用位置的限制。例如，可以在函数内部使用它们。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  var red = color.New(0xff0000)
  var green = color.New(0x00ff00)
  var blue = color.New(0x0000ff)

  ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  ...
}
```

</td></tr>
</tbody></table>

### Import Group Ordering
### import组内的包导入顺序

There should be two import groups:

应该有两类分组：

- Standard library
- 标准库
- Everything else
- 其他

This is the grouping applied by goimports by default.

默认情况下，这是 goimports 应用的分组。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Package Names
### 包名

When naming packages, choose a name that is:
当命名一个包时，请按照如下规则选择一个名称:

- All lower-case. No capitals or underscores.
- 全部小写，没有大写或者下划线。
- Does not need to be renamed using named imports at most call sites.
- 使用命名导入的情况下，大多数不需要重命名。
- Short and succinct. Remember that the name is identified in full at every call
  site.
- 短小简介。请记住，在每个使用的地方都完整标识了该名称。
- Not plural. For example, `net/url`, not `net/urls`.
- 不要复数。例如 `net/url` 而不是 `net/urls`。
- Not "common", "util", "shared", or "lib". These are bad, uninformative names.
- 不要用 “common”、“util”、“shared” 或 “lib”。它们都是不好而且信息不完备的名称。

See also [Package Names] and [Style guideline for Go packages].
参考 [Package Names] 和 [Style guideline for Go packages]。

  [Package Names]: https://blog.golang.org/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/

### Function Names
### 函数名称

We follow the Go community's convention of using [MixedCaps for function
names]. An exception is made for test functions, which may contain underscores
for the purpose of grouping related test cases, e.g.,
`TestMyFunction_WhatIsBeingTested`.

遵循Go社区使用 [MixedCaps for function names] 的约定。测试函数是个例外，为了对相关的测试用例进行分组，可能包含下划线，例如 “TestMyFunction_WhatIsBeingTested”。

  [MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

### Import Aliasing
### 导入别名

Import aliasing must be used if the package name does not match the last
element of the import path.

如果程序包名称与导入路径的最后一个元素不匹配，则必须使用导入别名。

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

In all other scenarios, import aliases should be avoided unless there is a
direct conflict between imports.

在所有其他情况下，除非导入之间有直接冲突，否则应避免导入别名。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Function Grouping and Ordering
### 函数分组与排序

- Functions should be sorted in rough call order.
- 函数应该粗略地以调用顺序排序。
- Functions in a file should be grouped by receiver.
- 在一个文件中的函数，应按接受者分组。

Therefore, exported functions should appear first in a file, after
`struct`, `const`, `var` definitions.

因此，导出的函数应先出现在文件中，放在 `struct`、`const`、`var` 定义的后面。

A `newXYZ()`/`NewXYZ()` may appear after the type is defined, but before the
rest of the methods on the receiver.

在定义类型之后，但在接收者的其余方法之前，可能会出现一个 `newXYZ()`/`NewXYZ()`。

Since functions are grouped by receiver, plain utility functions should appear
towards the end of the file.

由于函数是按接收者分组的，因此普通工具函数应在文件末尾出现。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reduce Nesting
### 减少嵌套

Code should reduce nesting where possible by handling error cases/special
conditions first and returning early or continuing the loop. Reduce the amount
of code that is nested multiple levels.

代码应通过尽可能先处理错误情况/特殊情况并尽早返回或继续循环来减少嵌套。减少嵌套了多个层级的代码量。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Unnecessary Else
### 不必要的 Else

If a variable is set in both branches of an if, it can be replaced with a
single if.

如果在if的两个分支中都设置了变量，则可以将其替换为单个if。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Top-level Variable Declarations
### 顶级变量声明

At the top level, use the standard `var` keyword. Do not specify the type,
unless it is not the same type as the expression.

在顶层，使用标准的 `var` 关键字。请勿指定类型，除非它与表达式的类型不同。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Specify the type if the type of the expression does not match the desired type
exactly.

如果表达式的类型与所需的类型不完全匹配，请指定类型。

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

### Prefix Unexported Globals with _
### 对于未导出的全局变量，使用_作为前缀

Prefix unexported top-level `var`s and `const`s with `_` to make it clear when
they are used that they are global symbols.

在未导出的顶级变量和常量， 前面加上前缀 `_`，以使它们在使用时明确表示它们是全局符号。

Exception: Unexported error values, which should be prefixed with `err`.

例外：未导出的错误值，应以err开头。

Rationale: Top-level variables and constants have a package scope. Using a
generic name makes it easy to accidentally use the wrong value in a different
file.

基本原理：顶级变量和常量具有包范围。使用通用名称可以轻松地避免在其他文件中意外使用错误的值。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

### Embedding in Structs
### 结构体的嵌入

Embedded types (such as mutexes) should be at the top of the field list of a
struct, and there must be an empty line separating embedded fields from regular
fields.

嵌入式类型（例如 mutexes）应位于结构的字段列表的顶部，并且必须有一个空行将嵌入式字段与常规字段分隔开。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

### Use Field Names to initialize Structs
### 使用字段名来初始化结构体

You should almost always specify field names when initializing structs. This is
now enforced by [`go vet`].

初始化结构时，应该尽可能始终指定字段名称。现在由 [`go vet`] 强制执行。

  [`go vet`]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Exception: Field names *may* be omitted in test tables when there are 3 or
fewer fields.

例外：如果字段少于3个，则字段可能会在测试表中省略。

```go
tests := []struct{
}{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

### Local Variable Declarations
### 本地变量声明

Short variable declarations (`:=`) should be used if a variable is being set to
some value explicitly.

如果将变量显式设置为某个值，则应使用简短的变量声明（`：=`）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

However, there are cases where the default value is clearer when the `var`
keyword is use. [Declaring Empty Slices], for example.

但是，在某些情况下，使用 `var` 关键字时默认值会更清晰。例如， [Declaring Empty Slices]。

  [Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### nil is a valid slice
### nil 是一个有效的分片

`nil` is a valid slice of length 0. This means that,

`nil` 是一个长度为 0 的分片。这意味着：

- You should not return a slice of length zero explicitly. Return `nil`
  instead.
- 你不需要显示地返回一个长度为 0 的切片。返回 `nil` 即可。

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- To check if a slice is empty, always use `len(s) == 0`. Do not check for
  `nil`.
- 检查切片是否为空，应该总是使用 `len(s) == 0`。而不用检查是否为 `nil`。

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- The zero value (a slice declared with `var`) is usable immediately without
  `make()`.
- 零值（用 `var` 声明的切片）可以立即使用，而无需 `make()`。

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

### Reduce Scope of Variables
### 缩小变量作用域

Where possible, reduce scope of variables. Do not reduce the scope if it
conflicts with [Reduce Nesting](#reduce-nesting).

尽可能缩小变量作用域。除非它与 [减少嵌套](#reduce-nesting) 冲突。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

If you need a result of a function call outside of the if, then you should not
try to reduce the scope.

如果需要在 if 之外获取函数调用的结果，则不应尝试缩小范围。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Avoid Naked Parameters
### 避免语意不明的参数

Naked parameters in function calls can hurt readability. Add C-style comments
(`/* ... */`) for parameter names when their meaning is not obvious.

函数调用中的裸参数可能会损害可读性。当参数名称的含义不明显时，添加类C语言的注释（`/ * ... * /`）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Better yet, replace naked `bool` types with custom types for more readable and
type-safe code. This allows more than just two states (true/false) for that
parameter in the future.

更好的是，使用自定义类型替换简易的 `bool` 类型来提高可读性和代码的类型安全。这样，将来该参数可以不仅仅两个状态（true/false）。

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Use Raw String Literals to Avoid Escaping
### 使用Raw String Literals，避免转义

Go supports [raw string literals](https://golang.org/ref/spec#raw_string_lit),
which can span multiple lines and include quotes. Use these to avoid
hand-escaped strings which are much harder to read.

Go 支持 [raw string literals](https://golang.org/ref/spec#raw_string_lit)，字符串可以跨越多行并包含引号。使用 Raw String Literals 可以避免难读的手工转义的字符串。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Initializing Struct References
### 初始化结构引用

Use `&T{}` instead of `new(T)` when initializing struct references so that it
is consistent with the struct initialization.

初始化结构引用时，请使用 `＆T{}` 而不是 `new(T)`，以使其与结构初始化一致。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Format Strings outside Printf
### 在 Printf 之外格式化字符串

If you declare format strings for `Printf`-style functions outside a string
literal, make them `const` values.

如果你为 `Printf`-style 函数声明格式字符串，请将格式化字符串放在外面，并将其设置为 `const` 常量。

This helps `go vet` perform static analysis of the format string.

这有助于 `go vet` 对格式字符串执行静态分析。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Naming Printf-style Functions
### 命名 Printf 样式的函数

When you declare a `Printf`-style function, make sure that `go vet` can detect
it and check the format string.

声明 `Printf`-style 函数时，请确保 `go vet` 可以检测到它并检查格式字符串。

This means that you should use pre-defined `Printf`-style function
names if possible. `go vet` will check these by default. See [Printf family]
for more information.

这意味着应该尽可能使用预定义的 `Printf` 样式的函数名称。 `go vet` 会默认检查这些内容。有关更多信息，请参见[Printf family]。

  [Printf family]: https://golang.org/cmd/vet/#hdr-Printf_family

If using the pre-defined names is not an option, end the name you choose with
f: `Wrapf`, not `Wrap`. `go vet` can be asked to check specific `Printf`-style
names but they must end with f.

如果不能使用预定义的名称，请以 f 结尾：`Wrapf`，而不是 `Wrap`。`go vet` 可以检查特定的`Printf`样式名称，但名称必须以 f 结尾。

```shell
$ go vet -printfuncs=wrapf,statusf
```

See also [go vet: Printf family check].
参
考 [go vet: Printf family check].

  [go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/

## Patterns
## 编程模式

### Test Tables
### Test 表

Use table-driven tests with [subtests] to avoid duplicating code when the core
test logic is repetitive.

在核心测试逻辑是重复的时，将表驱动测试与 [subtests] 配合使用可避免重复代码。

  [subtests]: https://blog.golang.org/subtests

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

Test tables make it easier to add context to error messages, reduce duplicate
logic, and add new test cases.

测试表使向错误消息添加上下文，减少重复的逻辑以及添加新的测试用例变得更加容易。

We follow the convention that the slice of structs is referred to as `tests`
and each test case `tt`. Further, we encourage explicating the input and output
values for each test case with `give` and `want` prefixes.

遵循这样的约定：将结构片称为“测试”，将每个测试用例称为“ tt”。此外，我们鼓励使用 `give` 和 `want` 前缀来说明每个测试用例的输入和输出值。

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

### Functional Options
### 功能选项

Functional options is a pattern in which you declare an opaque `Option` type
that records information in some internal struct. You accept a variadic number
of these options and act upon the full information recorded by the options on
the internal struct.

功能选项是一种模式，可以在其中声明一个不透明 Option 类型，该类型在某些内部结构中记录信息。接受这些选项的可变编号，并根据内部结构上的选项记录的全部信息采取行动。

Use this pattern for optional arguments in constructors and other public APIs
that you foresee needing to expand, especially if you already have three or
more arguments on those functions.

将此模式用于构造函数和您预期需要扩展的其他公共 API 中的可选参数，尤其是在这些函数上已经具有三个或更多参数的情况下。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

</td><td>

```go
type options struct {
  timeout time.Duration
  caching bool
}

// Option overrides behavior of Connect.
type Option interface {
  apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
  f(o)
}

func WithTimeout(t time.Duration) Option {
  return optionFunc(func(o *options) {
    o.timeout = t
  })
}

func WithCaching(cache bool) Option {
  return optionFunc(func(o *options) {
    o.caching = cache
  })
}

// Connect creates a connection.
func Connect(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    timeout: defaultTimeout,
    caching: defaultCaching,
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}

// Options must be provided only if needed.

db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
  addr,
  db.WithCaching(false),
  db.WithTimeout(newTimeout),
)
```

</td></tr>
</tbody></table>

See also,

参见：

- [Self-referential functions and the design of options]
- [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->
