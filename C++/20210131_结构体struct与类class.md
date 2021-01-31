# 结构体 struct 与 类 class
## 结构体 struct

结构体是C语言中的概念，它一种构造类型，可以包含若干成员变量，每个成员变量的类型可以不同；可以通过结构体来定义结构体变量，每个变量拥有相同的性质。例如：
```c++
#include <stdio.h>
//定义结构体 Student
struct Student{
    //结构体包含的成员变量
    char *name;
    int age;
    float score;
};
//显示结构体的成员变量
void display(struct Student stu){
    printf("%s的年龄是 %d，成绩是 %f\n", stu.name, stu.age, stu.score);
}
int main(){
    struct Student stu1;
    //为结构体的成员变量赋值
    stu1.name = "小明";
    stu1.age = 15;
    stu1.score = 92.5;
    //调用函数
    display(stu1);
    return 0;
}
```
运行结果：
```shell
小明的年龄是 15，成绩是 92.500000
```

## 类 class
C++ 中的类也是一种构造类型，但是进行了一些扩展，类的成员不但可以是变量，还可以是函数；通过类定义出来的变量也有特定的称呼，叫做“对象”。例如：
```c++
#include <stdio.h>
//通过class关键字类定义类
class Student{
public:
    //类包含的变量
    char *name;
    int age;
    float score;
    //类包含的函数
    void say(){
        printf("%s的年龄是 %d，成绩是 %f\n", name, age, score);
    }
};
int main(){
    //通过类来定义变量，即创建对象
    class Student stu1;  //也可以省略关键字class
    //为类的成员变量赋值
    stu1.name = "小明";
    stu1.age = 15;
    stu1.score = 92.5f;
    //调用类的成员函数
    stu1.say();
    return 0;
}
```
运行结果：
```shell
小明的年龄是 15，成绩是 92.500000
```

## 结构体struct与类class的对比

C语言中的 struct 只能包含变量，而 C++ 中的 class 除了可以包含变量，还可以包含函数。display() 是用来处理成员变量的函数，在C语言中，我们将它放在了 struct Student 外面，它和成员变量是分离的；而在 C++ 中，我们将它放在了 class Student 内部，使它和成员变量聚集在一起，看起来更像一个整体。

结构体和类都可以看做一种由用户自己定义的复杂数据类型，在C语言中可以通过结构体名来定义变量，在 C++ 中可以通过类名来定义变量。不同的是，通过结构体定义出来的变量还是叫变量，而通过类定义出来的变量有了新的名称，叫做对象（Object）。

在第二段代码中，我们先通过 class 关键字定义了一个类 Student，然后又通过 Student 类创建了一个对象 stu1。变量和函数都是类的成员，创建对象后就可以通过点号.来使用它们。

可以将类比喻成图纸，对象比喻成零件，图纸说明了零件的参数（成员变量）及其承担的任务（成员函数）；一张图纸可以生产出多个具有相同性质的零件，不同图纸可以生产不同类型的零件。

类只是一张图纸，起到说明的作用，不占用内存空间；对象才是具体的零件，要有地方来存放，才会占用内存空间。

在 C++ 中，通过类名就可以创建对象，即将图纸生产成零件，这个过程叫做类的实例化，因此也称对象是类的一个实例（Instance）。

有些资料也将类的成员变量称为属性（Property），将类的成员函数称为方法（Method）。