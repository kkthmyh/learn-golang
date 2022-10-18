#  Golang学习总结

##  包



Go的包主要有以下特性:

- 同一目录下的文件属于一个包，import时候会导入整个目录下的所有文件；
- package 名称不要求与目录名相同，即你可以在 **package1** 目录下将文件的 **package** 定义为**package2**
- 但同目录下的所有文件的 package 名称必须一致
- import 的时候可以给引入的包设置别名



##  访问控制

Go 跟 Java 类似，对于包内的函数，变量，struct，interface 等有访问控制，但不同的是，Go的访问控制并不是使用 public, protected, private 等关键字来实现 ，而是通过函数，变量，struct等名称的首字母是否大写来定义的，首字母大写则为 public，反之则为 private，没有 protected 这一级别。

```go
package funcation

import "fmt"

func Test() {
	fmt.Println("this is a public function")
}

func test() {
	fmt.Println("this is a private funcation")
}
```



##  继承

Go没有类似java的extends继承关键字，Go的继承如下

```go
package main

import (
	"fmt"
)

type parent interface {
	Hello() string
}

type child struct {
}

func (c *child) Hello() string {
	return "I'm child"
}
func getParent() parent {
	return new(child)
}
func main() {
	fmt.Println(getParent().Hello())
}

```

可以看到，**child** 并没有显性的继承 **parent**，它仅仅是实现了 **parent** 的 **Hello** 函数，即可在一个需要返回 **parent** 函数中作为返回值。



##  数组与切片

Go语言自带了两种集合数据结构，分别为数组和切片，其中给定了长度为数组，未给定长度即为切片

###  数组

数组示例：

```go
package main

import "fmt"

func main() {
	arr1 := [2]int{1, 2}
	arr2 := [...]int{1, 2, 3}
	fmt.Println(arr1)
	fmt.Println(arr2)
}

```



###  切片

切片示例：

切片类似于java中的ArrayList，数据结构如下

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

其中Data是一个指向切片所在数组的指针，也就是说切片实际上也是数组，如果使用不当可能会导致在总内存充裕的情况下由于连续内存不足而导致的意外错误。

要注意切片的扩容规则：

- 如果期望容量大于当前容量的两倍就会使用期望容量；
- 如果当前切片容量小于 1024 就会将容量翻倍；
- 如果当前切片容量大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；

切片示例：

```go
slice := []int{1, 2, 3}
slice := make([]int, 10)
```



## Map

Go的map和Java有所不同，用法如下：

```go
func main() {
	hash := map[string]int{
		"1": 1,
		"2": 2,
		"3": 3,
	}
	int1 := hash["1"]
	fmt.Println(int1)
}
```



##  for循环

###  常规用法

```go
for i := 0; i < 10; i++ {
	fmt.Println("2**%d = %d\n", i, v)
}
```

###  与range进行组合

在Go中，遍历数组，切片以及Map可以使用range来配合for循环使用：

数组/切片示例：

```go
arr1 := [3]int{1, 2, 3}
for index, num := range arr1 {
	fmt.Printf("%d at %d\n", num, index)
}
```

map示例：

```go
hash := map[string]int{
	"1": 1,
	"2": 2,
	"3": 3,
}
for k, v := range hash {
	fmt.Printf("%s: %d\n", k, v)
}
```



##  make和new

- make 的作用是初始化内置的数据结构，也就是我们在前面提到的切片、哈希表和 Channel
- new 的作用是根据传入的类型在堆上分配一片内存空间并返回指向这片内存空间的指针



##  panic和recover

Go没有try catch机制，普通异常使用返回 error 方式来处理，而严重异常则可以使用 panic 功能，但在实际使用中，因为无法预知用到的每个包是否有panic导致程序异常退出，则可使用 recover 功能拯救一下，panic 和 recover的完整示例如下：

```go
package main

import "fmt"

func panicedWithRecover() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
		fmt.Println("Recovered")
	}()

	panic("unknown err")

}
func panicedWithoutRecover() {
	panic("Another unknown err")

}
func main() {
	panicedWithRecover()

	fmt.Println("In main")

	panicedWithoutRecover()

	fmt.Println("In main again")
}
```

输出结果如下：

```go
unknown err
Recovered
In main
panic: Another unknown err

goroutine 1 [running]:
main.panicedWithoutRecover(...)
       ecover.go:17
main.main()
      recover.go:25 +0x9b
exit status 2
```

可以看到，如果在 defer 中调用 recover ，在 panic 后程序不会异常退出，但反之则退出了并抛出了错误。

