# Java语法
## jvm 和 jre 相关
### Java 语言编译过程
Java代码编译:是由Java源码编译器来完成;

Java字节码的执行:是由JVM执行引擎来完成



Java程序从源文件创建到程序运行要经过两大步骤：

1、源文件由编译器编译成字节码（ByteCode）

![](https://cdn.nlark.com/yuque/0/2024/png/26771572/1705133770047-576a9945-d2d4-402e-b1b8-a6da10a5d6c0.png)

2、字节码由java虚拟机解释运行。因为java程序既要编译同时也要经过JVM的解释运行，所以说Java被称为半解释语言（ "semi-interpreted" language）

![](https://cdn.nlark.com/yuque/0/2024/png/26771572/1705133685278-9d517c97-e33c-416b-9502-ad017b02ca9b.png)

```java
public class main{
	public void static main(){}
}
```

写这段代码，编译会过，但是 jvm 会找不到 main 入口

任何一个class一定要有 main 方法

![](https://cdn.nlark.com/yuque/0/2021/jpeg/12852882/1619778955130-9b14e8d3-d8bc-437a-8e78-4fb6cece1235.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_22%2Ctext_SGVhcnRMaW5rZWQ%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

## 命令行参数
Java 命令行参数`String []args`，`args[0]`是第一个参数，**不储存程序名字**。

```java
public class Main{
	public void static main(String []args){
        System.out.print(args[2]);
    }
}
// 命令行输入：
// java program_name hhhh gggg
// 输出：
// index out of bound exception
```

## 标志符
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697260497813-c9ef6c26-1b96-49d7-ac37-81fb71aee70c.png)

## 变量
在Java中，不区分变量的声明与定义

### 初始化
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697260621220-aac8a2b6-b7d4-4515-ae83-8d6732559b13.png)

局部变量不初始化使用，**<font style="color:#DF2A3F;">编译错误</font>**

**<font style="color:#DF2A3F;">但是static变量在全局，会初始化</font>**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697260865036-cfe07409-7292-4ae6-8972-cd0a70af93c1.png)

### 变量 type casting
+ byte 8位 1个字节
+ **char 16位 2个字节**
+ short 16位 2个字节
+ int 32位 4个字节
+ float 32位 4个字节
+ double 64位8个字节
+ long 64位 8个字节
+ **boolean 8位 1个字节**

**多个byte的变量都是用 big endian 储存的**

```java
public class Test {
    public static void main(String[] args) {
        int a = 2147483647*2;
        System.out.println(a); //-2   
        long b = 2147483647*2;
        System.out.println(b);//-2
    }
}

```

变量如果被赋值，超过所能够接受的范围，如：`char c = 1000;` 就**<font style="color:#DF2A3F;">编译错误</font>**

**<font style="color:#DF2A3F;">浮点数默认是double，</font>**`**<font style="color:#DF2A3F;">1.2f</font>**`**<font style="color:#DF2A3F;">或者</font>**`**<font style="color:#DF2A3F;">1.2F</font>**`**<font style="color:#DF2A3F;">表示float</font>**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697262064363-8da086a3-69b8-4bba-911e-c9b215804aff.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697262370124-f0923361-d479-4c70-a32b-abe30f58eff8.png)

Java**不支持自动**强制类型转换，如果需要，必须由程序**显式进行强制**类型转换。如上面的`int x = 5/2.0;`应该写成`int x = (int)(5/2.0)`

![](https://cdn.nlark.com/yuque/0/2024/png/26771572/1705134016219-b411a631-29de-4668-84b1-7a33994fa238.png)

赋值`float`需要`float a=1.0f`，`long`需要`long a = 1L`注意后缀！

### 常量声明用 final
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697261147041-6858c0e7-7bdb-420f-a30e-37e5422eb50c.png)

### final 用法
+ 修饰单个变量

**<font style="color:#DF2A3F;">但是如果final修饰数组，则数组里面的元素的值可以被改变</font>**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697261277960-469c423c-40ee-45f6-a260-e96066928e0e.png)

+ 修饰method

表示一个函数不可更改，也就是**不能被overload**，而不是修饰返回值的

+ 修饰class

表示整个类不能被继承了，自然，里面的所有方法也相当于被加了final。

+ 不能修饰interface

## 输入输出
### console
#### 输入
```java
import java.util.Scanner
public class Jy{
	public static void main(String[] args){
        Scanner input = new Scanner(System.in);
    }
}
```

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697261715197-13970f34-5e46-453e-8584-6209d031627c.png)

#### 输出
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697264533968-3d605f60-3dcc-47bc-b3fd-628065efb6d7.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697264587507-b84b87ec-3a9f-4824-9862-6d6d8f8a355b.png)

printf不按照格式来，编译能过但是运行挂

![](https://cdn.nlark.com/yuque/0/2024/png/26771572/1705134253378-31757036-cf10-467e-8ca8-5a7011209763.png)

![](https://cdn.nlark.com/yuque/0/2024/png/26771572/1705135699615-f4ec16ba-b321-4265-b952-9a867ef75cb9.png)

### file
## switch 
只能对`String int short byte char `选择

```java
switch(exp){
    case value$1:
        ...
        break;
    case value$2:
        ...
        break;
    default:
        break;
}
```

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697262881567-77010c5e-2c0f-44fc-bfac-c90e3c42d452.png)

## Math class
### 舍入
`ceil`天花板，向上取，`floor`地板，向下取

`rint`x is rounded to its **nearest integer**. If x is equally close to **two** integers, the **even** one is returned as a double.

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697263083860-3b9e089f-1a83-41d3-8094-1f600d8fee5b.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697263247481-46180782-7143-4c1b-9259-18daa61e233b.png)

### random()
`Math.random()`左闭右开`[0,1)`

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697263552905-d2e9dc1a-fe7d-42b4-b74c-6c950ccd52ca.png)

## Character class
注意这些**都有返回值**，要改变某个变量的值，需要`char c = toLowerCase(c);`而不是`toLowerCase(c);`

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697263698408-30d53635-22a1-45ca-a469-62c9b94dfe6a.png)

## String class
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697263924585-d975eaad-e053-4ca9-8677-27e85c208370.png)

### 比较
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697264244434-8e40231a-9081-4f36-a152-f6533a5c18c7.png)

### 子串处理
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697264325473-c73cac80-c1d5-414c-881e-5f9e305cee9a.png)

`substring(begin,end)`**左闭右开**`[begin,end)`

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697264392410-0c3bcecd-1cff-40d3-861c-752edd7bd3e8.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697264453149-cb287ec1-44d5-4e29-9c33-c0c2f2826afb.png)

### 正则表达式匹配字符串
TODO

### .format()方法
TODO

### String.valueOf()方法
TODO

## 一维数组
Java数组分配在堆上

命令行参数，带有String[] args在java中，args[0]是第一个参数，程序名**没有**存储在args中

### 只声明
```java
int[] myarray;
int myarray[];// allowed
```

### 只创建
```java
myList = new double[10];
```

创建完数组后有默认初始值

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697265221045-4ee4ac6e-d55d-44a7-abee-ed89f7f21b6a.png)

### 声明+创建
```java
double myList[] = new double[10];
```

### 初始化
```java
// 声明+create+初始化一起，只能写成一句
double[] myList = {1.9, 2.9, 3.4, 3.5};//此时 [] 紧跟着datatype

// 等价：
// 声明+创建
double[] myList = new double[4];
// + 初始化
myList[0] = 1.9;
myList[1] = 2.9;
myList[2] = 3.4;
myList[3] = 3.5; 
```

### copy
```java
System.arraycopy(sourceArray, 0, targetArray, 0, sourceArray.length);


// 或者用Arrays的copyOf函数

int[] copiedLuckyNumbers = Arrays.copyOf(luckNumbers,luckyNumbers.length);
// 第2个参数是新数组的长度，这个方法通常用来增加数组的大小：
int[] copiedLuckyNumbers = Arrays.copyOf(luckNumbers,2*luckyNumbers.length);
// 如果数组元素是数值型，则多余的元素被赋值为0；若是布尔型，则赋值为false。
// 相反，如果长度小于原始数组长度，则只拷贝前面的元素。
```

### Arrays.binarySearch Method
+ 找到，返回下标
+ 找不到，返回：`-被查找元素本应该出现的下标-1`

```java
// For the binarySearch method to work, the array must be pre-sorted in increasing order. 
int[] list = {2, 4, 7, 10, 11, 45, 50, 59, 60, 66, 69, 70, 79};
System.out.println("Index is " + java.util.Arrays.binarySearch(list, 11));
// Return is 4

char[] chars = {'a', 'c', 'g', 'x', 'y', 'z'};
System.out.println("Index is " + java.util.Arrays.binarySearch(chars, 't'));
// Return is –4 (insertion point is 3, so return is -3-1)
```

### Arrays.sort Method
```java
double[] numbers = {6.0, 4.4, 1.9, 2.9, 3.4, 3.5};
java.util.Arrays.sort(numbers);
char[] chars = {'a', 'A', '4', 'F', 'D', 'P'};
java.util.Arrays.sort(chars);
```

**匿名数组**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697265729786-a0082ba1-d67f-44cb-991e-d96a28f288b8.png)

函数返回一个数组

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697265828060-a87082a3-39dd-4128-91fe-3425625568a5.png)

## 多维数组
### 只声明
```java
int[][] myarray;
int myarray[][];// allowed
```

### 只创建
```java
myList = new double[10][10];
```

### 声明+创建
```java
double myList[][] = new double[2][10];
```

### 初始化
```java
// 声明+create+初始化一起，只能写成一句
double[][] myList = {{1.9, 2.9}, {3.4, 3.5}};//此时 [] 紧跟着datatype

// 等价：
// 声明+创建
double[][] myList = new double[2][2];
// + 初始化
myList[0][0] = 1.9;
myList[0][1] = 2.9;
myList[1][0] = 3.4;
myList[1][1] = 3.5; 
```

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697270195057-29a29dc5-9cd2-41bb-97cb-7d381879d7d7.png)

### length
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1697270257206-7cd5b301-d078-4752-ae32-3ca4bb5cf2f1.png)

matrix.length 是行数量

# Java 面向对象
## 类 class
一个Java源文件，可以包含多个类，但是只能包含一个公共类。

Java 中用变量定义状态，用方法定义行为

+ Java 中的类也有构造函数，在对象构造的时候调用，但是对象的定义也可以不需要构造函数，当类定义里没有显式声明的构造函数时Java 编译器会自动调用一个default constructor
+ Java 中方法和成员变量的引用方式：objectRefVar.methodName(arguments)

– Reference Data Fields 有默认值

∗ String 类型是null，数值类型是0，布尔类型是false，char 类型是\u0000 ，但是Java 对于方法中的局部变量不会赋予默认值

∗ 比如`int x` 后直接对其进行print 会发生编译错误，因为变量没有初始化

### main 方法
在Java中，`main()`方法是Java应用程序的入口方法，也就是说，程序在运行的时候，第一个执行的方法就是`main()`方法，这个方法和其他的方法有很大的不同，比如方法的名字必须是`main`，方法必须是`public static void` 类型的，方法必须接收一个字符串数组的参数等等。

```java
public class HelloWorld2 {
    static {
        System.out.println("Hello Wordld!");
    }
    public static void main(String args[]){
        System.exit(0);
    }
}

输出 Hello Wordld!
```

### instance 和staic——和C++ 相同
+ instance 是变量的实例，instance variable 属于特定的实例，instance method 由类的一个实例调用
+ **<font style="color:#DF2A3F;">static 类型的变量和方法被一个类的所有成员共享</font>**

## 值传递和引用传递
Passing by value for primitive type value (the value is passed to the parameter)

Passing by value for reference type value (the value is the **reference** to the object)

+ Java 本质上是一种值传递

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700396147774-34e5a5ab-887e-4fab-aa6d-7707d939ce98.png)

<font style="color:#DF2A3F;">这个程序</font>**<font style="color:#DF2A3F;">无法交换a和b的值</font>**<font style="color:#DF2A3F;">。因为只是拷贝了a和b的引用，在</font>`<font style="color:#DF2A3F;">swap</font>`<font style="color:#DF2A3F;">函数里面对引用进行交换。</font>

+ 对象中的一个数组实际上是一个reference variable 的数组
+ [https://www.cnblogs.com/ming-szu/p/9165393.html](https://www.cnblogs.com/ming-szu/p/9165393.html)  **<font style="color:#DF2A3F;">Java对object数组排序 特别注意lambda表达式，上机考试可以用</font>**

### package相关的访问权限
By default, the class, variable, or method can be accessed by **any class in the same package**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700395588522-60eb171b-ffdd-42f7-a723-897737a82617.png)

+ `private`修饰：只能单个class内部访问
+ ` `没有修饰：能在单个package里访问
+ `public`修饰：所有都可以访问

#### import package注意事项
+ `import`**不会递归**，只会引入当前package下的直接类。

比如 `import java.util.*`，只会引入`java.util`下的直接类，而不会引入`java.util`下嵌套包的类，如**不会**引入`java.util.zip`下的类。试图嵌套引入的形式也是无效的，如import java.util.*.*

+ import不仅可以导入类，还可以导入**静态方法和静态域**。

比如：`import static java.lang.System.*`;就可以使用System类的静态方法和静态域了。				`out.println(“Hello World!”);  //System.out`

`exit(0);    //System.exit`

+ 需要将类文件切实安置到其所归属之Package所对应的相对路径下。

`package com.horstmann.core` 

`public class Employee{...}`，需要将Employee.java放到`com/horstmann/core`目录下，编译器也会将class文件放到相同的目录结构中

## 不可变类 
+ `final` class / 让类的所有构造器都变为私有的或包级私有的，并添加公共的静态工厂来代替公有的构造器。
+ 所有数据域都是`private`
+ 没有`set`方法，对象的引用不会泄露

## 初始化
总结起来说，java中数据成员的初始化过程是：

①先默认初始化

②进行定义处的初始化（指定初始化）

③构造函数初始化

## 装箱
### autoboxing
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700398345259-2a9edb37-61c4-4c6d-9343-185107eb1e08.png)

### <font style="color:#DF2A3F;">Numeric Wrapper Class--BigInteger & BigDemical</font>
#### .valueOf() 方法 与 .intVaule() 
**<font style="color:#DF2A3F;">valueOf会调用缓存机制(如果有实现)，并且返回一个object</font>**

```java
Double doubleObject= Double.valueOf("12.4");
Integer integerObject= Integer.valueOf("12");
int tmp = (new Integer(3432)).intValue();
```

#### .toString()方法
```java
BigInteger num = new BigInteger("34532");
String ss = num.toString();
```

`<font style="color:rgb(0, 0, 0);">public int compareTo(BigDecimal val)</font>`<font style="color:rgb(0, 0, 0);">比较大小（仅比较两个BigDecimal对象数值是否相等，忽略精度；大于则返回正数，小于则返回负数，等于返回0）</font>

`<font style="color:rgb(0, 0, 0);">public BigDecimal max(BigDecimal val)</font>`<font style="color:rgb(0, 0, 0);">返回较大值</font>

## 常量池
![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700398781717-916ad0ea-6edc-49ae-a015-9c6ede0b7c4a.png)

例子：

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700399042868-50c1f4f9-a6d7-4fab-84f8-916b4b2a6f3d.png)

结果应该如下：

```java
true
true
false
false
false
true//不要求intern
```

java中基本类型的包装类的大部分都实现了常量池技术，这些类是Byte,Short,Integer,Long,Character,Boolean

**两种浮点数类型的包装类则没有实现**。

Byte,Short,Integer,Long,Character这5种整型的包装类也只是**对从-128 到127的对象使用对象池**，也即对象不负责创建和管理大于127和小于-128的这些类的对象。利用缓存机制实现常量池：为了减少不必要的内存消耗和内存开辟次数，Integer 里做了一个缓存，缓存了从-128 到127 之间的Integer 对象，总共是**256**个对象。

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700399418128-2a2b6a49-8e0a-4760-896e-ca54f415844c.png)

结果应该如下：

```java
// i11 调用 Integer.valueOf(126)
// i12 调用 Integer.valueOf(126)
// i13 调用 new Integer(126)
// i14 调用 Integer.valueOf(126)
// i15 调用 Integer.valueOf(125+(new Integer(1)).intValue)
true
false
true
true
// 128 不在缓存范围内
false
false
false
false
```

**区分原始数据类型和类**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700399930396-8e22627d-2f7e-4e1f-88a8-40c96feb2e95.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700400075858-b38f1da8-f894-4516-bc62-088d225420a1.png)

**浮点数包装类没有实现缓存**

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700400006128-9347ea17-db48-4e2e-bc34-149fd13e4e10.png)

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1700398991651-75d85e60-b431-4a07-8a7c-29be2343d1e0.png)

## 枚举类
采用`enum`关键字定义，所有的枚举类型都继承自`Enum`类型，通常常量用`public final static` 来定义，在枚举类中可以用如下方式定义

```java
public enum Light{
	public final static RED = 1, GREEN = 2, YELLOW = 3;
}
```

### 枚举类的特性
+ 枚举类是`final`，**不能被继承**
+ 含有`values()`的静态方法，可以按照声明顺序返回其值**数组**
+ `ordinal()` 方法，返回枚举值在枚举类中的顺序，根据声明时候的顺序决定(**从0开始**)
+ 用`valueOf()` 来得到枚举实例，`toString()` 将枚举转化为可以打印的字符串
+ 枚举类型支持switch

```java
public enum Enumty {
    MON, TUE, WED, THU, FRI;
    public static void main(String[] args) {
        for (Enumty e : Enumty.values()) {
            System.out.println(e + ":" + e.toString() + ":" + e.ordinal() + ":" + e.name());
        }
    }
}
输出结果：
MON:MON:0:MON
TUE:TUE:1:TUE
WED:WED:2:WED
THU:THU:3:THU
FRI:FRI:4:FRI
```

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1698828017506-b72d5f95-2bff-4f67-bfdc-3bc0219424d3.png)

输出结果解释：

+ 星期.天.名 输出了枚举常量`天`的属性`名`，即 "星期天"。
+ 星期.天 输出了枚举常量`天`的名字，即 "天"。
+ 天.名 输出了枚举常量`天`的属性`名`，即 "星期天"。
+ 天 输出了枚举常量`天`的名称，即 "天"。

• Java 的枚举本质上是int 值

–

### 在枚举类加入方法，域
在枚举类中，可以将不同的行为与每个常量关联起来。例如，对于Operation中的PLUS, MINUS, TIMES和DIVIDE，有不同的操作。

```java
public enum Operation{
    PLUS, MINUS,TIMES,DIVIDE
    double apply(double x, double y){
        switch(this){ // 注意写法 
        case PLUS:
            return x+y;
        case MINUS:
            return x-y;
        case TIMES:
            return x*y;
        case DIVIDE:
            return x/y;
        }
        throw new AssertionError("Unknown operation:"+this);
    }
}
```

改一下：

```java
public enum Operation{
    PLUS{
        double apply(double x, double y){
     	   return x+y;
        }
    },
    MINUS{
        double apply(double x, double y){
   		     return x-y;
        }
    };
	abstract double apply(double x, double y);
}
```

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1698829962828-90860d32-559d-41c0-bb61-0fbbda5ee413.png)

```java
4 PLUS 2  6
4 MINUS 2 = 2
4 TIMES 2 = 8
4 DIVIDE 2 = 2
```

程序填空

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1698830100705-a634803e-e348-43f1-bb51-2bdf87c5ab43.png)

## 继承
Java 中继承的关键词是`extends`

`super()`：

+ 子类没有继承基类的构造函数，但是子类中可以用关键字super 去调用基类的构造函数

父类有无参构造器，子类才可以写无参构造器；父类有含参构造器，子类才可以写含参构造器.

下面的执行顺序是，先执行父类`A`，先去执行`int i`的构造，再执行A的无参，再执行B的i，最后B的无参。同名的函数使用的是B的。

```java
class A 
{ 
    private int i=baz(); 
    public int baz() { 
        System.out.print("A"); 
        return 0; 
    } 
    A()
    {
        System.out.println("Base A");
    }
} 
class B extends A 
{ 
    private int i=baz(); 
    public int baz() { 
        System.out.println("B"); 
        return 10;
     } 
    public static void main(String[] args) { 
        A a = new B();
    }
    B()
    {
        System.out.println("Child B");
    }
}
```

![](https://cdn.nlark.com/yuque/0/2023/png/26771572/1698830872089-2c16dcf2-545b-4a65-bffa-9dbb284d6792.png)

B继承了A，但是B的构造里面没有调用A的构造

  TODO override 和 继承的区别

