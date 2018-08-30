---
description: Go语言支持一些面向对象的特征。方法就是其中之一。方法是一种特殊的带有接收参数的函数。这篇文章将介绍Go方法相关的概念。
---

# 方法

### 方法声明

在Go语言中，我们可以显式声明类型 T 和 \*T 的方法，类型 T 必须满足4个条件：

1. T 必须是一个已经被定义的类型。
2. T 的定义和方法声明必须在同一个包中。
3. T 不能是指针类型。
4. T 不能是接口类型。

T是所有声明为T和\*T的方法的基础接收类型。

我们还可以声明T和\*T类型别名的方法。这些被声明的方法与 T， \*T的方法效果相同。

如果为一个类型声明了一个方法，我们可以称之为这个类型拥有这个方法。

从上面列出的条件，可以得出结论我们不能声明为如下声明方法：

* 内建基础类型，例如 int 和 string，因为我们无法在内置标准包中声明方法。
* 接口类型。但是接口类型可以拥有方法。
* 未命名的类型除了指针类型 \*T，包括未命名的数组，map，切片，函数，channel还有结构体类型。然而，如果一个未命名的struct类型内嵌其它有方法的类型，编译器会为此未命名struct隐式声明一些方法。未命名的指针类型的基础类型是未命名的struct类型。

方法声明类似函数声明，但是它一个额外的参数声明部分。这个额外参数声明被称为接收参数。每个方法必须声明仅一个接收参数。

这里有一些方法声明的例子：

```go
// Age 和 int 是两种不同的类型
type Age int

func (age Age) LargerThan(a Age) bool {
    return age > a
}

func (age *Age) Increase() {
    *age++
}

// 接收者是被定义的函数类型
type FilterFunc func(in int) bool 

func (ff FilterFunc) Filte(in int) bool {
    return ff(in)
}

//接收者是被定义的Map类型
type StringSet map[string]struct{}

func (ss StringSet) Has(key string) bool {
    _, present := ss[key]
    return present
}

func (ss StringSet) Add(key string) {
    ss[key] = struct{}{}
}

func (ss StringSet) Remove(key string) {
    delete(ss, key)
}

//接收者是被定义的结构体类型
type Book struct {
    pages int
}

func (b Book) Pages() int {
    return b.pages
}

func (b *Book) SetPages(pages int) {
    b.pages = pages
}
```

从上面的例子，我们可以知道接收者基础类型不仅可以是结构体类型，还可以是其它类型，例如基础类型和容器类型，只要接受者类型满足上面的4个条件。

在其它编程语言中，接受者参数通常被隐式定义为this，但在Go语言中不推荐使用this作为接收者参数的名字。

\*T被称为指针接收者，非指针接收者被称为值接收者。就个人而言，我不建议将术语指针视为术语值的反面，因为指针只是特殊的值。但是，我不反对在这里使用术语指针接收者和值接收者。

方法的名字可以是空标识符 \_，一种类型可以存在多个名字是空标识符的名字。但是这些方法不会被调用。

只有可以导出的方法可以被其它包调用。

### 每个方法对应一个隐式函数

对于每个方法声明，编译器都会隐式声明一个对应的函数。比如在上一节被声明的两个方法，编译器隐式声明如下两个函数

```go
func Book.Pages(b Book) int {
    return b.pages
}

func (* Book).SetPages(b *Book, pages int) {
    b.pages = pages
}
```

在这两个隐式声明中，接收者参数被移除然后作为被插入正常参数的第一个位置。函数体和方法体是相同的。

这隐式方法名，Book.Pages 和 （\*Book）.SetPages，都是 TypeDenotation.MethodName 这种形式。由于Go中的标识符不能包含句点特殊字符，这两个隐式方法名是非法标识符，因此这两个方法不能被显示声明。它们只能被编译器隐式声明，但是它们可以被用户代码调用：

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) Pages() int {
	return b.pages
}

func (b *Book) SetPages(pages int) {
	b.pages = pages
}

func main() {
	var book Book
	
	(*Book).SetPages(&book, 123)
	fmt.Println(Book.Pages(book))
```

### 指针接收者的隐式方法

对于每个声明给值接收者T的方法，编译器都会对应声明一个相同的方法名的隐式函数给\*T。通过上面的例子，方法Pages是针对Book类型声明的，因此编译器针对\*Book类型隐式声明一个相同的方法Pages。同名方法只包含一行代码，这是对上面介绍的隐式函数Book.Pages的调用。

```go
func (b *Book) Pages() int {
    return Book.Pages(*b)
}
```

这就是为什么我不建议将术语指针视为术语值的反面。毕竟，当我们为一个非指针类型显式声明一个方法时，实际上2个方法被声明。

正如上一节提及的，每个被声明的方法，编译器都会对应声明一个隐式函数。因此对于刚刚提及的隐式方法，编译声明一个隐式函数如下。

```go
func (*Book).Pages(b *Book) int {
    return Book.Pages(*b)
}
```

换句话说，值接收者每显式声明一个方法，同时会隐式声明2个函数和1个方法。

### 方法原型和方法集



