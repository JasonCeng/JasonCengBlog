# golang omitempty 总结

在使用Golang的时候，不免会使用Json和结构体的相互转换，这时候常用的就是 `json.Marshal`和`json.Unmarshal`两个函数。

这时候在定义json结构体的时候，我们会用到omitempty这个字段，这个字段看似简单，但是却有很多小坑，这篇文章带你稍微研究一下他的用途和功能

Basic Usage

当我们设置json的struct的时候，会定义每个字段对一个json的格式，比如定义一个dog 结构体：
```golang
type Dog struct {
	Breed string
	WeightKg int
}
```

现在我们对他进行初始化，将其编码为JSON格式：
```golang
func main() {
	d := Dog{
		Breed:    "dalmation",
		WeightKg: 45,
	}
	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
则输出的结果为：{"Breed":"dalmation","WeightKg":45},你可点击这里.

现在假如有一个结构体变量我们没初始化，那么结果可能也会跟我们预期的不太一样：
```golang
func main() {
	d := Dog{
		Breed:    "pug",
	}
	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
输出的结果为：{"Breed":"pug","WeightKg":0},明显dog的weight是未知，而不是0,并不是我们想要的结果，我们更想要的结果是："WeightKg":null。

为了实现这样的目的，我们这时候应该使用`omitempty`变量来帮我们实现，当我们在Dog结构体加上这个tag的时候：
```golang
type Dog struct {
	Breed    string
	// The first comma below is to separate the name tag from the omitempty tag 
	WeightKg int `json:",omitempty"`
}
```
输出结果为：{"Breed":"pug"}。现在WeightKg就被设置为默认零值（对于int应该为0，对于string应该为""， 指针的话应该为nil）。

不能单纯使用omitted

当结构体相互嵌套的时候，那么omitempty就可能出现问题，比如：
```golang
type dimension struct {
	Height int
	Width int
	}

type Dog struct {
	Breed    string
	WeightKg int
	Size dimension `json:",omitempty"`
}

func main() {
	d := Dog{
		Breed: "pug",
	}
	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
输出结果为：

{"Breed":"pug","WeightKg":0,"Size":{"Height":0,"Width":0}}
我们已经使用omitempty标注的dimension还是显示了出来。这是因为结构体dimension不知道空值是什么，GO只知道简单结构体例如int,string,pointer 这种类型的空值，为了不显示我们没有提供值的自定义结构体，我们可以使用`结构体指针`：
```golang
type Dog struct {
	Breed    string
	WeightKg int
	// Now `Size` is a pointer to a `dimension` instance
	Size *dimension `json:",omitempty"`
}
```
运行结果为：

{"Breed":"pug","WeightKg":0}

为什么会这样呢？因为指针是基本类型，Golang知道他的空值是啥，所以就直接赋值为nil(指针类型的空值)。

现在出一个问题，下面的程序的初始结果是什么？
```golang
type Dog struct {
	Age *int `json:",omitempty"`
}

func main() {
	age := 0
	d := Dog{
		Age: &age,
	}

	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
A. {"Age":0}

B. {}

答案在最后揭晓。

0, "", nil 三者的区别

现在很难的区别的是：零值，值为0，默认值。

比如我们设置一个restaurant结构体：
```golang
type Restaurant struct {
	NumberOfCustomers int `json:",omitempty"`
}

func main() {
	d := Restaurant{
		NumberOfCustomers: 0,
	}
	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
他的输出结果为：

{}
这肯定不是我们想要的结果，因为这个时候我们想要他输出0，他却没有任何值输出。因为Golang把0当成了零值，所以跟没有赋值是一样的，比如：
```golang
type Restaurant struct {
	NumberOfCustomers int `json:",omitempty"`
	Name              string
}

func main() {
	d := Restaurant{
	   //给NumberOfCustomers赋值
		NumberOfCustomers: 0,
		Name:              "hello",
	}
	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
和
```golang
type Restaurant struct {
	NumberOfCustomers int `json:",omitempty"`
	Name              string
}

func main() {
	d := Restaurant{
	    //未给NumberOfCustomers赋值
		Name: "hello",
	}
	b, _ := json.Marshal(d)
	fmt.Println(string(b))
}
```
二者结果都是：{"Name":"hello"}

解决这样的问题的一种方法是使用int指针，因为int指针的空值为nil,当我想输出0的时候，我传进去地址，地址肯定不是空值nil，这样肯定会显示出来0.
```golang
type Restaurant struct {
	NumberOfCustomers *int `json:",omitempty"`
}

func main() {
	d1 := Restaurant{}
	b, _ := json.Marshal(d1)
	fmt.Println(string(b))
	//Prints: {}
	
	n := 0
	d2 := Restaurant{
		NumberOfCustomers: &n,
	}
	b, _ = json.Marshal(d2)
	fmt.Println(string(b))
	//Prints: {"NumberOfCustomers":0}
}
```
基于上，我们应该谨慎使用omitempty,如果选择使用了他，那你就要第一时间知道，各个值的空值是什么？当我没有给某个变量赋值的时候，他应该是什么样的，我想要什么的输出？这都是你要仔细斟酌的。

好了，现在公布答案：A:因为他是int类型的指针，我们传进去的也是指针，所以不会有任何问题。同时&age不是指针的nil值，所以不会被忽略，显示的时候不会有问题，就是0。

## 参考
[1] https://www.cnblogs.com/yuanmh/p/14116384.html