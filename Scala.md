# Scala

## 第一章 Scala入门

### 1.1 Scala概述

- Scala全称Scalable Language（可伸缩语言），是一门完成的软件编程语言
- Scala是完全面向对象的编程语言
- 作者：Martin Odersky
- Scala源码经过编译后会生成和Java编译后相同的字节码文件，因此可以被JVM解析执行



### 1.2 Scala安装

1. 解压scala压缩包，配置bin目录到系统环境变量

2. 启动IDEA，安装scala插件
   ![image-20200518214930349](Scala.assets/image-20200518214930349.png)

3. 创建Maven工程，添加Scala库

   ![image-20200518215112956](Scala.assets/image-20200518215112956.png)![image-20200518215451442](Scala.assets/image-20200518215451442.png)
   
4. 创建Object型的Scala Class，编写如下代码并执行

   ```scala
   Object Test {
       def main(args : Array[String]) : Unit = {
           println("hello scala")
       }
   }
   ```



## 第二章 变量和数据类型

### 2.1 注释

- Scala的注释使用与Java完全一致

  ```scala
  //单行注释
  
  /*
  多行注释
  */
  
  /**
  文档注释
  */
  ```



### 2.2 变量

- 声明语法

  ```scala
  var/val 变量名 : 变量类型 = 变量值
  
  //举例
  var age : Int = 18
  val name : String = "lcx"
  ```

- 变量的初始化：与Java不同，Scala中的变量必须在声明时就初始化赋值

  ```scala
  var age1 : Int //错误
  var age2 : Int = 18 //正确
  ```

- 可变变量（var）：变量初始化后可以重新赋值

  ```scala
  var age2 : Int = 18
  ```

- 不可变变量（val）：变量初始化后不可以再赋值，类似于Java中的final

  ```scala
  //对于引用类型变量,不可变的是引用,即地址值,而属性值是可以修改的
  val name : String = "lcx"
  ```



### 2.3 标识符

- 标识符的命名规则

  1. 可以使用大小写字母、数字、大部分符号作为标识符
  2. 不能以数字开头
  3. 尽量避免使用$开头，可能和某些符号在编译后的结果冲突
  4. 不能与关键字或保留字相同，一定要用时，可以加飘号``

- 标识符的命名规范：同Java

- 由于Java标识符不能使用$和_以外的符号，因此Scala中额外支持的符号会在编译时进行转义

  ```scala
  //举例 :-> 在编译后会转换为$colon$minus$greater
  ```
  
- **能声明的标识符不一定能使用**，不能使用时需要根据实际情况再修改



### 2.4 字符串

- scala中没有定义String类，而是直接使用了Java中的String类，因此也保留了String类对象的**不可变性**

- 字符串连接

  ```scala
  val name : String = "lcx"
  val info : String = "name = " + name
  val forString : String = "for" * 3 //forforfor
  println(info)
  ```

- 传值字符串（printf）

  ```scala
  val name : String = "lcx"
  printf("name = %s", name)
  //注意printf中,传入的参数数量必须大于等于%s的数量,否则运行报错,大于时后续参数不被使用
  ```

- 插值字符串

  ```scala
  val name : String = "lcx"
  println(s"name = $name")
  println(s"name = ${name.substring(0,1)}")
  //s的作用为对字符串中的$转义,表示变量名
  ```

- 多行字符串

  ```scala
  //使用三个连续"封装字符串,每行以|顶格符开始,不再需要转义内部的"
  //常用于封装JSON和SQL
  val json =
  	s"""
  		|{"username" = "$name", "age" = $age}
  		|""".stripMargin
  println(json)
  ```




### 2.5 输入与输出

- 读取控制台的输入

  ```java
  //Java中的读取方式
  Scanner scan = new Scanner(System.in);
  int i = scan.nextInt();
  ```

  ```scala
  //Scala中的读取方式
  val name : String = StdIn.readLine()
  val age : Int = Std.readInt()
  ```

- 读取文件输入

  ```java
  //Java中的读取方式
  BufferedReader br = new BufferedReader(new FileReader("input/inputtest"));
  String line;
  while ((line = br.readLine()) != null){
      System.out.println(line);
  }
  br.close();
  ```

  ```scala
  //Scala中的读取方式
  val iterator : Iterator[String] = Source.fromFile("input/inputtest").getLines()
  while (iterator.hasNext){
      println(iterator.next())
  }
  ```

- 输出

  ```java
  //Java中的输出方式
  BufferedWriter bw = new BufferedWriter(new FileWriter("output/outputtest"));
  bw.write("hello java");
  bw.close();
  ```

  ```scala
  //Scala中的输出方式
  val writer = new PrintWriter("output/outputtest")
  writer.println("hello scala")
  writer.close()
  ```

- 网络

  ```scala
  //服务端
  object Server {
    def main(args: Array[String]): Unit = {
      val serverSocket = new ServerSocket(9526)
      val socket = serverSocket.accept()
      val line : Int = socket.getInputStream.read()
      println(line)
    }
  }
  ```

  ```scala
  //客户端
  object Client {
    def main(args: Array[String]): Unit = {
      val socket = new Socket("localhost",9526)
      socket.getOutputStream.write(123)
      socket.close()
    }
  }
  ```



### 2.6 数据类型

#### 2.6.1 Java的数据类型

- 基本数据类型（8种）：byte，short，int，long，float，double，char，boolean
- 引用数据类型：Object，String，包装类，数组，集合等等

#### 2.6.2 Scala的数据类型

- Scala是完全面型对象的数据类型，因此没有基本数据类型，只分**值对象类型**和**引用对象类型**
- 值对象类型
  - **AnyVal：所有值对象类型的父类**
  - Byte，Short，Int，Long，Float，Double，Char，Boolean
  - **Unit：类似于Java中的void，表示无返回值的结果类型，仅有一个实例“()”**
- 引用对象类型
  - **AnyRef：所有引用对象类型的父类**
  - Array，String
  - 所有的Java类
  - 其他的Scala类
  - **Null：表示空的引用，唯一实例null，可以赋值给所有其他的引用对象类型**
- **Any类：Scala中所有类的父类**
- Nothing类：Scala中所有类的子类



### 2.7 类型转换

#### 2.7.1 自动类型转换

```scala
//自动类型提升规则如下
//Byte -> Short -> Int -> Long -> float -> Double
//Char -> Int -> Long -> float -> Double
val b : Byte = 10
val s : Short = b
val i : Int = s
val i : Double = 'A'
//注意:以下代码可正确运行(计算发生在编译阶段,不涉及自动类型转换)
val c : Char = 'A' + 1
```

#### 2.7.2 强制类型转换

```java
//Java中的强制类型转换
int i = 10
byte b = (byte)i
```

```scala
//Scala中的强制类型转换,使用类中提供的类型转换方法强制转换,转换的原理与Java相同
val i : Int = 10
val b : Byte = i.toByte
```

#### 2.7.3 字符串的类型转换

```scala
//和Java相同,scala中也为所有类提供了toString()方法
val i : Int = 10
val s : String = i.toString
//也可以使用字符串拼接的方式转换
val s : String = i + ""
```



## 第三章 运算符

### 3.1 算数运算符

| +    | -    | *    | /    | %    |
| ---- | ---- | ---- | ---- | ---- |
| 加   | 减   | 乘   | 除   | 取余 |



### 3.2 关系运算符

| ==   | !=     | >    | <    | >=       | <=       |
| ---- | ------ | ---- | ---- | -------- | -------- |
| 等于 | 不等于 | 大于 | 小于 | 大于等于 | 小于等于 |

```scala
//在Java中, == 用于比较引用数据类型时,比较的是二者是否指向内存中的同一个对象
//在Scala中, == 实际上相当于先对二者进行非空判定,再调用了equals方法比较值是否相等
val a = new String("abc")
val b = new String("abc")
println(a == b)  //true
println(a.equals(b))  //true
println(a eq b)  //false,eq方法比较的是二者的地址值
```



### 3.3 赋值运算符

| 运算符 | 举例                      |
| ------ | ------------------------- |
| =      | C = A + B                 |
| +=     | C += A 等价于 C = C + A   |
| -=     | C -= A 等价于 C = C - A   |
| *=     | C *= A 等价于 C = C * A   |
| /=     | C /= A 等价于 C = C / A   |
| %=     | C %= A 等价于 C = C % A   |
| <<=    | C <<= A 等价于 C = C << A |
| >>=    | C >>= A 等价于 C = C >> A |
| &=     | C &= A 等价于 C = C & A   |
| ^=     | C ^= A 等价于 C = C ^ A   |
| \|=    | C \|= A 等价于 C = C \| A |



### 3.4 逻辑运算符

| &&     | \|\|   | !      |
| ------ | ------ | ------ |
| 逻辑与 | 逻辑或 | 逻辑非 |

- **Scala中逻辑运算符两侧必须为Boolean型变量或常量，不能出现计算式**

  ```java
  //以下代码在java中可以运行,但Scala中不能通过编译
  Boolean b = false;
  System.out.println(true || (b=true));
  System.out.println(b);
  ```



### 3.5 位运算符

| 运算符 | 作用说明                       | 举例  | 结果 |
| ------ | ------------------------------ | ----- | ---- |
| <<     | 左移，空位补0                  | 3<<2  | 12   |
| >>     | 右移，正数空位补0，负数空位补1 | 3>>1  | 1    |
| >>>    | 无符号右移，空位补1            | 3>>>1 | 1    |
| &      | 与，有0则0，全1则1             | 6&3   | 2    |
| \|     | 或，有1则1，全0则0             | 6\|3  | 7    |
| ^      | 异或，相同为0，不同为1         | 6^3   | 5    |
| ~      | 取反，1取0，0取1               | 6     | -7   |



### 3.6 运算符本质

- **scala中实际是没有运算符的，所有的运算符本质上都是方法**
- 调用运算符方法时，“.”可以省略
- 调用运算符方法时，若形参数量<=1，可以省略“.”和"()"

```scala
//举例:计算2+3
//完整的写法
val i = 2.+(3)
//省略写法
val i = 2 +(3)
val i = 2 + 3
```



## 第四章 流程控制

### 4.1 if-else 分支控制

- Scala中的分支控制语句的语法格式与Java相同。

- **Scala中的分支条件相比于Java要求更加严格，不能使用如（b = true）一类的表达式，这是因为在Scala中，表达式本身是属于Unit类型的，必须使用Boolean类型对象作为分支条件。**

- **Scala中的分支控制语句具有返回值，返回值为分支语句中的最后一行语句的返回值，返回值类型为所有分支返回值类型的最小共同父类**

  ```scala
  val age = 25
  if (age < 15){
      println("少年")
  }else if (age < 30){
      println("青年")
  }else if (age < 55){
      println("中年")
  }else {
      println("老年")
  }
  //if-else语句的返回值可声明变量接收,变量的类型为所有分支返回值类型的最小共同父类
  val info = if (age < 15){
      "少年"
  }else if (age < 30){
      "青年"
  }else if (age < 55){
      "中年"
  }else {
      "老年"
  }
  println(info)
  ```



### 4.2 循环控制

#### 4.2.1 for循环

- 基本语法

  ```scala
  //循环1-5,包含边界
  for (i <- 1 to 5){
  	println(i)
  }
  
  //循环1-5,不包含5
  for (i <- Range(1,5)){
      println(i)
  }
  
  //循环1-5,不包含5
  for (i <- 1 until 5){
      println(i)
  }
  ```

- 循环守卫，相当于在当次循环开始时增加循环条件，用于判断当次循环是否执行

  ```scala
  for (i <- 1 to 5 if i != 3){
      println(i)
  }
  //等价于
  for (i <- 1 to 5){
      if (i != 3){
          println(i)
      }
  }
  ```

- 循环步长，设定循环条件参数的迭代方式，默认为+=1

  ```scala
  //方式1:使用by关键字设置增幅,适用于所有循环条件集表示方式
  for (i <- 1 to 5 by 2){
      println(i)
  }
  //可以设置负数实现循环参数从大到小迭代
  for (i <- 5 to 1 by -1){
      println(i)
  }
  
  //方式2:通过Range方法参数设置增幅
  for (i <- Range(1, 5, 2)){
      println(i)
  }
  //可以设置负数实现循环参数从大到小迭代
  for (i <- Range(5, 1, -1)){
      println(i)
  }
  ```

- 循环嵌套

  ```scala
  //方式1:和Java中的语法规则相同
  for (i <- 1 to 5){
      for (j <- 1 to 4){
          println(i + " " + j)
      }
  }
  //方式2:Scala中提供了另一种方式实现和上述循环相同的功能
  for (i <- 1 to 5; j <- 1 to 4){
      println(i + " " + j)
  }
  
  //虽然方式2可以简化了嵌套for循环的编写,但在使用上具有局限性,无法实现内外层循环之间的代码逻辑
  //示例:使用一次for循环输出以下图形
          *
         ***
        *****
       *******
      *********
     ***********
    *************
   ***************
  *****************
  for (i <- 1 to 9){
      println(" " * (9 - i) + "*" * (2 * i - 1))
  }
  ```

- 引入变量：循环条件中可以引入变量

  ```scala
  for (i <- 1 to 5; j = i - 1){
      println(j)
  }
  //效果与以下方式相同
  for (i <- 1 to 5){
      val j = i - 1
      println(j)
  }
  ```

- 循环的返回值：与分支控制语句相同，for循环语句也是具有返回值的

  ```scala
  //默认的返回值类型为Unit
  val test = for (i <- 1 to 3) {
      "atguigu"
      i*3
  }
  println(test) //输出()
  
  //使用yield关键字可以接收每次循环的最后一行语句的返回值
  val test = for (i <- 1 to 3) yield {
      "atguigu"
      i*3
  }
  println(test) //输出(3,6,9)
  ```



#### 4.2.2 while循环

```scala
//Scala中while循环的语法与Java完全一致,返回值类型为Unit
var i = 1
while(i < 5) {
    println(i)
    i += 1
}

//Scala中do-while循环的语法与Java完全一致,返回值类型为Unit
var i = 1
do {
    println(i)
    i += 1
}while(i < 5)
```



#### 4.2.3 break退出循环

- 不同于Java中循环中使用的break和continue关键字，Scala中取消了这两个关键字，break的功能由新的函数实现

  ```scala
  //使用Break类的breakable()抓取break()抛出的异常,将整个循环作为参数传入
  Breaks.breakable(
  	for (i <- 1 to 5) {
  		if (i == 4){
               //使用Break类的break()结束循环,底层的实现方式为抛出异常
  			Breaks.break()
  		}
  		println(i)
  	}
  )
  ```



## 第五章 函数式编程

### 5.1 基础函数编程

#### 5.1.1 函数定义基本语法

```scala
修饰符 def 函数名(形参列表) : 返回值类型 = {
    函数体
}
//示例
private def test(s : String) : Unit = {
    println(s)
}
```



#### 5.1.2 函数与方法

- Scala中，方法与函数的定义语法完全相同，方法是函数的一种
- 区别
  1. **直接定义在类中的函数即为方法**
  2. **方法具有重载和重写的特性，函数没有**
  3. **函数支持嵌套声明，方法不支持**



#### 5.1.3 函数的定义

```scala
//1.无参无返回值
def func1() : Unit = {
    println("func1")
}
//2.无参有返回值
def func2() : String = {
    "func2"
}
//3.带参无返回值
def func3(s : String) : Unit = {
    println("func3" + s)
}
//4.带参有返回值
def func4(s : String) : String = {
    "func4" + s
}
//5.多参无返回值
def func5(s : String, i : Int) : Unit = {
    println("func5" + s + i)
}
//6.多参有返回值
def func6(s : String, i : Int) : String = {
    "func6" + s + i
}
```



#### 5.1.4 函数的参数

- Java与Scala的对比

  - 在Java中，方法的形参在传入后可以在方法内修改值或是引用，也可以使用final关键字修饰形参使其不可变。
  - 在Scala中，方法的形参传入后都是不可变的。

- 可变形参

  ```scala
  //和Java中的可变形参一样,Scala中也支持传入不确定数量的形参,使用*表示
  def test(str : String*) : Unit = {
      println(str)
  }
  //注意:可变形参必须作为形参列表的最后一个形参声明
  ```

- 参数的默认值

  ```scala
  //Scala中可以定义形参的默认值,当未传入该参数时,使用默认值赋值形参
  def test(name : String, password : String = "000000"): Unit ={
      println(name + " " + password)
  }
  test("lcx") //输出 lcx 000000
  test("lcx", null) //输出 lcx null
  ```

- 带名传参

  ```scala
  //Scala提供了带名的传参方式,在为函数传入参数时可以不遵循定义的顺序
  def test(password : String = "000000", name : String): Unit ={
      println(name + " " + password)
  }
  test(name = "lcx", password = "123456")
  ```



#### 5.1.5 至简原则

- 为了提升开发效率，scala在编译时会进行全面的动态判定，使函数的代码得到了最大程度的简化

  ```scala
  //原函数
  def test(s : String) : String = {
      return "lcx"
  }
  
  //1.当return语句为函数体的最后一行代码返回值时,return关键字可以省略
  def test(s : String) : String = {
      "lcx"
  }
  
  //2.当函数返回值类型可以自动推断,:和返回值类型可以省略
  def test(s : String) = {
      "lcx"
  }
  //此时必须省略return关键字,因为return关键字会唯一确定返回值类型,无法触发类型推断
  
  //3.当函数代码逻辑只有1行时,{}可以省略
  def test(s : String) = "lcx"
  
  //4.当函数的形参列表为空时,()可以省略
  def test = "lcx"
  //若函数定义时未省略(),则可以使用test或test()调用函数
  //若函数定义时省略了(),则只能使用test()调用函数
  
  //5.由于返回值类型为Unit类型的函数无法通过自动类型推断生成,当需要省略:Unit时,可以省略=
  def test(s : String) {
      return "lcx"
  }
  //此时{}无论如何不能省略
  ```
  
- 匿名函数：只关心代码逻辑，不关心函数名称的函数

  ```scala
  //格式 (参数列表) => {函数体}
  () => println("lcx")
  //使用时可赋值给变量,再用变量进行调用
  val a = () => println("lcx")
  a()
  
  //注意
  //1.匿名函数在声明时必须赋值给变量或作为形参传入其他函数,不能只声明不使用
  //2.匿名函数的参数列表和=>可以省略,使用_代替多个形参,前提条件为每个形参被顺序调用且仅被调用一次
  (s1 : String, s2 : String) => s1 + s2
  _ + _
  ```



### 5.2 进阶函数编程

#### 5.2.1 函数作为值

```scala
//函数整体可以作为值赋值给变量,有以下两种情况

//1.定义函数时有形参列表
def test01() = "lcx"
	//此时函数的多种引用赋值方式意义如下
	test01 //函数返回值,可以赋值给未指定类型的变量,类型为String,也可以赋值给类型为() => String的变量,表示函数整体
	test01() //函数返回值,类型为String
	test01 _ //函数整体,类型为() => String,可以直接赋值给未指定类型的变量
	test01() _ //函数返回值,类型为String,下划线无意义

//2.定义函数时省略形参列表
def test02 = "lcx"
	//此时函数的多种引用赋值方式意义如下
	test01 //函数的返回值,类型为String,可以赋值给未指定类型的变量
	test01() //错误的引用方式
	test01 _ //函数整体,类型为() => String,可以赋值给未指定类型的变量
	test01() _ //错误的引用方式,下划线也救不了

//总结
//1.将函数的返回值赋值给变量,无参函数直接使用"函数名",带参函数使用"函数名(参数)"
val a = test
//2.将函数整体赋值给变量,无脑使用"函数名 _"
val a = test _
```



#### 5.2.2 函数作为参数

```scala
//函数整体可以作为形参传递给其他函数
def func1(i : Int) : Int = {
    i * 2
}
def func2(f : (Int => Int), n : Int) : Int = {
    f(n)
}
println(func2(func1, 10))

//注意:使用函数赋值给形参时,同样遵循函数的值传递机制,因此最保险的传入方式还是使用"函数名 _"
func2(func1 _, 10)
```



#### 5.2.3 函数作为返回值

```scala
//函数可以作为返回值,通常同于嵌套函数
def fun() : (Int) => Int = {
    def test(i : Int) = {
        i * 2
    }
    test _
}
//整体函数作为返回值时,返回值与声明的返回值类型关系,与函数赋值给变量的关系一致
	//1.当返回的函数在声明时省略了形参列表时,必须使用"函数名 _"的方式返回
	//2.当返回的函数在声明时含有形参列表,则可以使用_触发自动类型推断,也可以注明返回的函数类型
```



#### 5.2.4 匿名函数

- 匿名函数赋值给变量

  ```scala
  //使用时可赋值给变量,再用变量进行调用
  val a = () => println("lcx")
  a()
  ```

- 匿名函数作为参数传递给函数

  ```scala
  //参数为函数的函数
  def fun(f : (Int) => Int) = {
      f(10)
  }
  
  //传入匿名函数调用函数
  fun( (x : Int) => { x * 10 } )
  //1.当参数类型可以自动推断,且参数在函数体中按顺序仅调用一次时,匿名函数可以进一步简化，()=>全部省略
  fun(_ * 10)
  //2.当函数体与参数无关时,可以使用_代替形参列表
  fun(_ => 5)
  //3.不能直接以下划线作为函数
  fun(_) //错误
  ```



#### 5.2.5 闭包

- 在嵌套函数中，内部函数可能会调用到外部函数中的变量，此时该变量会作为额外的参数传入内部函数，使得即使外部函数执行完成出栈后，变量也得以保留，这一过程称为闭包。（Scala-2.12）

- 在Scala-2.11及之前的版本中，闭包的实现方式是将外部函数中的变量在编译时作为匿名函数类中的属性存储。

- 调用示例

  ```scala
  def test(i : Int) : Int => Int = {
      def sum(j : Int) : Int = {
          i + j
      }
      sum _
  }
  test(10)(20) //输出30
  ```



#### 5.2.6 函数柯里化

- 柯里化：将原来具有n个参数的函数，改写为具有n层嵌套结构的函数的过程

- 本质：外部函数传入第一个参数，内部函数传入其余参数，并将内部函数作为整体返回给外部函数，以此类推

- 具体实现

  ```scala
  //原函数
  def test(i : Int, j : Int) : Int = {
      i * j
  }
  
  //柯里化后的函数
  def test(i : Int)(j : Int) : Int = {
      i * j
  }
  //柯里化的本质
  def test(i : Int) : Int => Int = {
      def test1(j : Int) : Int = {
          i * j
      }
      test1 _
  }
  
  //柯里化后的相关数据类型
  test _		// Int => Int => Int
  test(10)	// Int => Int
  test(10)(5)	// Int
  ```



#### 5.2.7 控制抽象

- Scala中，可以将一整段代码逻辑作为参数传递给函数，代码逻辑的类型为 => Unit，返回值类型为Unit

  ```scala
  def test(op : => Unit) : Unit = {
      op
  }
  //调用函数
  test(
      println("atguigu")
  )
  ```



#### 5.2.8 递归函数

- 普通递归

  ```scala
  //示例:计算参数的阶乘
  def test(n : Int) : Int = {
      if (n == 1){
          1
      } else {
          n * test(n -1)
      }
  }
  //1.递归函数必须声明返回值类型
  //2.代码逻辑中调用自身,并设置跳出递归的逻辑
  //3.普通递归在每次调用递归函数时,都需要等待下一层递归的结果,导致函数的在栈中大量积压,容易出现栈溢出
  ```

- 尾递归

  ```scala
  //示例:计算参数的阶乘
  def test(n : Int, result : Int) Int = {
      if (n == 1){
          result
      } else {
          test(n - 1, result * n)
      }
  }
  //1.递归函数的执行不依赖于外部函数,每一层递归在执行结束后立刻出栈
  //2.调用自身的代码在递归中必须作为最后一行执行
  //3.底层的执行方式为循环
  ```



#### 5.2.9 惰性函数

- 在把函数执行的返回值赋值给变量时，有时需要占用大量内存，而变量并不需要立即调用，造成资源的浪费

- 可以通过lazz关键字，将函数的执行延后到变量的首次调用

  ```scala
  def test() : String= {
      println("test...")
      "lcx"
  }
  lazy val a = test()
  println("----------")
  println(a)
  ```



## 第六章 面向对象编程

### 6.1 面向对象编程基础

#### 6.1.1 包

- 基本语法（与Java一致）

  ```scala
  package com.atguigu.scala
  //与Java中包的声明不同的是,scala中的包的逻辑路径完全决定于包的声明语句,与包的物理存储地址无关
  ```

- 扩展语法

  1. 包名可多次声明或嵌套使用

     ```scala
     //省略{}多次声明
     package com
     package atguigu
     package scala
     
     //嵌套声明
     package com {
         package atguigu {
             package scala {   
                 //代码逻辑
             }
         }
     }
     
     //以上两种声明方式等同于 package com.atguigu.scala
     ```

  2. 子包中的类可以直接访问同一源码文件中父包的内容（Java中不同包无论父子关系都需要import）

     ```scala
     package com
     package atguigu {
         class Test {}
         package scala {
             val test = new Test
         }
     }
     ```

  3. **可以创建包对象（package object），包对象中的属性方法可以被包下所有类访问，1个包只能有1个包对象**

     ```scala
     package com.atguigu
     //包对象名默认使用包名
     package object scala {
     	val name = ""
         def test() = {
             println("package object")
         }
     }
     ```

     

#### 6.1.2 导入

- 基本语法（与Java中的导入语法相同，使用全类名导入指定类）

  ```scala
  //导入包下指定类
  import com.atguigu.scala.Test
  //导入包下所有类需要使用_代替Java中的*
  import com.atguigu.scala._
  
  // 以下3个包中的类会默认导入,无需显式声明
  // java.lang._ 
  // scala._ 
  // scala.Predef._ 
  ```

- 扩展语法

  1. Scala中的import语句支持在任意位置使用

     ```scala
     object Test {
         import java.sql.Date
         val date = new Date(1995,2,6)
     }
     ```

  2. 可以导包，不指定具体类

     ```scala
     import java.util
     val list = new ArrayList()
     ```

  3. 可以一次导入一个包中的多个指定类

     ```scala
     //使用{}和,声明
     import java.util.{List,ArrayList}
     ```

  4. 可以屏蔽包中的指定类，解决不同包下同名类的冲突

     ```scala
     //导入java.util包中除ArrayList类外的所有类,必须使用{}
     import java.util.{ArrayList=>_, _}
     ```

  5. 可以在导入类时起别名

     ```scala
     import java.sql.{Date=>SqlDate}
     val date = new SqlDate(1995,2,6)
     ```

  6. 可以指定从根目录导入类，默认的导入路径是从当前包下开始的相对路径

     ```scala
     //指定从根目录下导入类
     import _root_.com.atguigu.scala
     ```
     
  7. 可以导入伴生对象和伴生类的对象
  
     ```scala
     //1.声明伴生类和伴生对象
     class ImportTest {
         def classFun() = {
             println("class")
         }
     }
     object ImportTest {
         val t = 1
         def objFun() = {
             println("object")
         }
     }
     
     //2.导入伴生对象或伴生对象中的属性,导入后伴生对象的属性方法可以直接调用
     import ObjTest
     import ObjTest.t
     fun()
     
     //3.导入class类对象,导入后对象的属性方法可以直接调用(必须为val对象,var对象不可导入)
     val a = new ClassTest
     import a
     fun()
     ```



#### 6.1.3 类

- 在Java中，一个类中可以声明静态或非静态的属性方法，而静态语法不符合scala完全面向对象的编程思想，为此scala将类的静态和非静态结构进行了拆分。实现的方式是**将静态和非静态结构分别封装在同名的object和class类中**，其中object类通过单例对象实现了类静态语法。

- object类

  - 封装静态的属性和方法，类中声明的属性方法可以使用类名即伴生对象直接调用

  - object类在加载时会生成一个单例对象（伴生对象），通过伴生对象实现了完全面向对象的静态语法

  - 与object类同名的class类称为该object类的伴生类

    ```scala
    object objTest {
        var name : String = "atguigu"
        def fun() = {
            println("objTest")
        }
    }
    //1.直接使用类名访问伴生对象的属性方法
    objTest.name = "atguigu"
    objTest.fun()
    //2.直接使用类名获取伴生对象后调用
    val a = objTest
    a.name = "atguigu"
    //类的伴生对象在内存中有且仅有1个,以下二者输出结果相同
    println(objTest)
    println(a)
    ```

- class类

  - 封装非静态的属性方法，类中声明的属性方法必须通过实例化对象调用

    ```scala
    class objTest {
        var age = 25
    }
    //必须通过实例对象调用类中的结构,调用类的无参构造器时可省略()
    val a = new objTest()
    ```



#### 6.1.4 属性

- 基本语法

  ```scala
  class User {
      //不同于Java中的成员变量有默认的初始化值
      //Scala中声明的变量必须显式赋值,可使用_给属性赋值为相应数据类型的默认值
      var name : String = _
      var age : Int = _
  }
  ```

- 扩展语法

  1. Scala中声明一个类的属性时，不仅仅声明了属性本身，还会自动编译生成类似于Java中的get、set方法用于访问

  2. Scala中属性本身的权限为private，而权限修饰符实际修饰的是两个访问方法的权限

  3. 类get方法名与属性名一致

  4. 类set方法名为“属性名_$eq”，val关键字定义的属性没有该方法

  5. 由于Scala中的get、set方法命名没有遵循Java中的Bean规范，导致在很多现有框架中无法调用类的属性，因此Scala提供了@BeanProperty标签，实现Scala中类的属性对Bean规范的支持。

     ```scala
     @BeanProperty
     var name : String = _
     @BeanProperty
     var age : Int = _
     ```



#### 6.1.5 权限修饰符

- Scala中的四种权限修饰符：与Java不同，scala中提供了私有、包私有、受保护、公共四种访问权限

| 访问权限         | 同类 | 同包及其子包 | 子类 | 同工程 |
| ---------------- | ---- | ------------ | ---- | ------ |
| private          | √    |              |      |        |
| private[package] | √    | √            |      |        |
| protected        | √    |              | √    |        |
| (default)        | √    | √            | √    | √      |

- 权限修饰符的使用
  
```scala
  //第一个private表示类私有,第二个private表示类的主构造方法私有
  private class User private{}
```

- private
  声明为private的属性的两个get、set方法也是私有的，并且无法使用@BeanProperty标签

- private[package]
  声明的package可以是所在类的任意直接或间接父包，指定父包及其子包中的所有位置都可以访问该属性

- protected

  明确属性方法的**提供者和调用者**

  ```scala
  package com.atguigu.scala
  object Field {
      def main(args: Array[String]): Unit = {
          val test : FieldTest = new FieldTest
          //调用Object中clone()的方法,权限为protected
          test.clone()
          //方法的提供者 java.lang.Object
          //方法的调用者 com.atguigu.scala.Field
          //提供者与调用者无子父类关系,因此无法调用执行
      }
  }
  class FieldTest {}
  ```

- (default)
  任意位置都可以调用



#### 6.1.6 方法

- 方法的声明

  - 方法声明在类中，声明方式与函数完全相同

- 默认包含的方法

  ```scala
  val user = new User
  //1.java.lang.Object类中的方法
  user.toString
  user.hashCode
  //2.scala中的方法
  user.isInstanceOf[User] //判断user是否为User类的对象,伴生对象会返回false
  user.asInstanceof[User] //将user装换为User类的对象
  val clazz : Class[User] = classOf[User] //获取对象的类型信息
  ```

- apply()方法

  ```scala
  //在object类中声明apply()方法,可以通过伴生对象构建伴生类对象
  object User {
      def apply() = {
          new User
      }
  }
  //不同的对象获取方式对比
  val user1 = new User()		//使用伴生类构造方法创建伴生类对象
  val user2 = User.apply()	//使用伴生对象的apply方法创建伴生类对象
  val user3 = User()			//同上,省略apply
  val user4 = User			//获取伴生对象
  
  //在class类中也可以声明apply方法,通过实例对象调用
  val user = new User()
  user()
  ```

- 方法的重载（overload）

  - 基本规则与Java一致
  - **当调用方法的参数满足多个形参具有子父类关系的重载方法时，系统会执行形参为最小子类（最接近类）的方法**
  - **当调用方法的参数满足多个形参为值类型的重载方法时，系统会执行形参精度最接近的方法**

- 方法与属性的重写（override）

  - 方法的重写

    1. 在方法声明时使用override关键字重写，**不能重写private权限的方法**
  2. 重写的方法返回值类型必须与原方法返回值类型相同或是其子类
    3. 重写方法的访问权限不能小于原方法

  - 属性的重写

    1. 在属性声明时使用override关键字重写，**不能重写private权限的属性**
  2. **scala中不允许在子类中定义和父类中同名的属性，但可以使用override重写val属性**
    3. 因为scala中访问父类的属性会默认调用属性的get,set方法，而方法的动态绑定会调用子类的重写方法，访问子类中重写的属性，这种结果与初衷不符产生歧义，所以禁止var属性的重写
    4. 若重写后的属性需要使用@BeanProperty标签，重写前的属性也需要添加该标签
    5. **重写后的属性不能使用_进行默认赋值**
  
  - 多态与虚拟方法的调用
  
    ```scala
    object MethodTest {
        def main(args: Array[String]): Unit = {
            val p : Person = new User
            p.fun //输出结果为40
        }
    }
    
    class Person{
        val i : Int = 10
        def fun = {
            println(i + 10)
        }
    }
    
    class User extends Person{
        //重写父类中的属性
        override val i : Int = 20
        //重写父类中的方法
        override def fun = {
            println(i + 20)
        }
    }
    ```



#### 6.1.7 构造方法

- 主构造方法

  - 定义：Scala中，类本身也是一个函数，定义时可以使用()声明形参列表，这个函数就是类的主构造方法。

  - 主构造方法的形参列表中，**可以使用var/val将参数直接定义为类的属性**，略去了形参赋值给属性的过程。

  - 没有定义为var/val的参数也会被编译为类的私有属性，但该属性没有get/set方法，无法从外部访问。

    ```scala
    //声明Person类,name属性直接通过形参赋值
    class Person(val name : String){
        var age = _
    }
    ```

- 辅助构造方法

  - 定义：定义在类中的构造方法，用于与实现Java中构造器重载的相同功能

  - 辅助构造方法的方法名为this

  - 辅助构造方法的首行必须直接或间接地调用主构造方法

    ```scala
    //声明私有化主构造器的Person类
    class Person private(val name : String){
        var age = _
        def this(name : String, age : Int = 18){
            this(name)
            this.age = age
        }
    }
    ```

- 子类的构造方法

  ```scala
  //父类
  class Person(val name : String, val age : Int){}
  //子类的主构造方法在继承父类时可以声明参数,系统会根据参数自动调用父类中相应的构造方法
  class User(name : String, age : Int) extends Person(name, age){}
  ```



### 6.2 面向对象编程进阶

#### 6.2.1 继承

- 和Java中相同，Scala中也使用extends关键字实现类的继承，同样类的继承也必须满足单继承机制。

  ```scala
  class Person {}
  class User extends Person{}
  ```



#### 6.2.2 封装

- 在Scala中，属性在声明的同时会自动进行封装，将属性本身私有化，并提供不同权限的访问方法。



#### 6.2.3 抽象

- 抽象类

  - 抽象属性和抽象方法

    ```scala
    //抽象属性:只声明,不初始化的属性.
    var name : String
    
    //抽象方法:只声明,不实现的方法.不显式声明返回值类型时,默认为Unit.
    def test1()
    def test2() : String
    ```

  - 具有抽象属性或抽象方法的类必须声明为抽象类，抽象类可以不具有抽象属性或抽象方法

    ```scala
    abstract class AbstractTest{
        var name : String
        def test1()
    }
    ```

  - 抽象类的对象无法直接构建，需要通过具体的实现类构建

- 抽象类的实现类

  - 抽象类的继承类必须重写所有的抽象属性和方法，否则继承类仍然需要定义为抽象类

    ```scala
    abstract class AbstractTest{
        var name : String
        def fun()
    }
    //使用extends实现抽象类
    class Test01 extends AbstractTest{
        var name : String = ""
        def fun() : Unit = {}
    }
    ```

  - 重写的使用区别

    1. 抽象属性方法的重写**可以省略override关键字**，成员属性方法的重写必须使用override关键字
    2. **var/val的抽象属性都支持重写**，只有val的成员属性支持重写
    3. 抽象属性重写后可以直接使用@BeanProperty注解，成员属性重写后必须保证和重写前属性同步使用注解
    4. 声明为var的属性在重写时可以使用_赋默认值，**声明为val的属性在重写时不能使用\_赋值**



#### 6.2.4 单例对象

- 在scala中，object类在加载时会默认产生一个单例对象，即伴生对象

- object类只会在内存中加载一次，同时初始化一个伴生对象，因此可以使用object类的特性在scala中方便的实现java中的单例模式

  ```scala
  //1.私有化伴生类的构造器
  class Single private {}
  //2.在伴生对象中提供获取单例对象的方法
  object Single {
      val single = new Single
      def apply() = {
          Single
      }
  }
  //3.获取单例对象
  val a = Single()
  val b = Single()
  println(a)
  println(b)
  ```



#### 6.2.5 特质

- 特质的说明

  1. Scala将多个类的相同特征剥离后组成一个独立的语法结构，称为特质（trait）
  2. 特质是与类并列的结构，可以继承类与其他特质，也可以被类或其他特质继承
  3. 特质支持多继承，一个类或特质可以继承多个特质
  4. 特质中可以声明抽象属性与方法，并且实现的规则与抽象类的实现规则一致

- 特质的混入

  ```scala
  //方式1:类声明时使用extends和with关键字
  trait Trait01{}
  trait Trait02{}
  trait Trait03{}
  class Test01 extends Trait01 with Trait02 with Trait03
  //extends可继承父类/特质,继承多个特质时使用with
  
  //方式2:调用类的构造方法时使用with动态混入特质
  trait Trait01{}
  trait Trait02{}
  class Test01{}
  val test = new Test with Traint01 with Traint02
  ```
  
- 特质的初始化过程（从父到子，从左到右）

  ```scala
  object Initialize01 {
      def main(args: Array[String]): Unit = {
          //判断初始化过程
          new SubUser01 //abdce
      }
  }
  //声明一个特质和两个子特质
  trait Test01 {println("a")}
  trait SubTest01 extends Test01 {println("b")}
  trait SubTest02 extends Test01 {println("c")}
  //声明具有父子关系的类并分别继承特质
  class User01 extends SubTest01 {println("d")}
  class SubUser01 extends User01 with SubTest02 {println("e")}
  
  //初始化总结:从父到子,从左到右
  //1.当前类初始化前,需要先初始化继承的父类/特质
  //2.继承多个类/特质时,从左到右依次初始化
  ```

- 特质的方法调用过程

  ```scala
  object Trait01 {
      def main(args: Array[String]): Unit = {
          val mysql0 = new Mysql01
          mysql0.fun() //输出顺序为ccc bbb aaa
      }
  }
  
  trait Operate01{
      def fun(): Unit = println("aaa")
  }
  
  trait DB01 extends Operate01 {
      override def fun(): Unit = {
          println("bbb")
          super.fun()
      }
  }
  
  trait Log01 extends Operate01 {
      override def fun(): Unit = {
          println("ccc")
          super.fun()
      }
  }
  
  class Mysql01 extends DB01 with Log01{}
  
  //总结:
  //特质中的super不代表直接父类,而是指上一级特质(继承列表中的前一个特质)
  //若需要调用指定特质中的方法,可以使用super[类名]的方式调用指定类中的方法
  ```



#### 6.2.6 扩展

- 枚举类

  ```scala
  object EnumTest {
      def main(args: Array[String]): Unit = {
          println(Color.RED.id)	//1
          println(Color.RED)	//red
      }
  }
  //1.定义object类继承Enumeration抽象类
  object Color extends Enumeration{
      //2.使用Value方法定义枚举类的对象
      val RED = Value(1,"red")
      val YELLOW = Value(2,"yellow")
      val BLUE = Value(3,"blue")
  }
  ```

- 应用类

  ```scala
  //1.定义object类继承App特质
  object AppTest extends App{
  	//2.直接在类中声明执行代码,相当于执行main方法
  }
  ```

- 数据类型的别名

  ```scala
  //使用type关键字为指定数据类型起别名
  type s = String
  val name : s = "atguigu"
  ```



## 第七章 集合

### 7.1 集合的概述

- Scala中集合分为三大类：序列Seq，集Set，映射Map
- 所有的集合都扩展自Iterable特质
- Scala中每一个集合的实现类都提供了可变和不可变两个版本
  - 可变集合所在包：scala.collection.mutable
  - 不可变集合所在包：scala.collection.immutable



### 7.2 Array数组

- Scala中的两种数组
  - 不可变数组（Array）：数组一旦初始化，索引不可变，因此数组长度不可变
  - 可变数组（ArrayBuffer）：数组初始化后可以动态调整数组长度

#### 7.2.1 不可变Array

- 基本语法

  ```scala
  //1.创建数组
  val array = new Array[String](5) //使用new创建长度为5的String型数组
  val array = Array(1,2,3,4,5) //使用伴生对象的apply方法创建数组对象
  
  //2.数组元素赋值
  array(0) = "a" //使用索引赋值
  array.update(0,"a") //使用update方法赋值
  
  //3.遍历数组
  array.foreach(println) //使用自定义方法遍历
  println(array.mkstring(",")) //使用mkstring方法遍历
  
  //4.创建多维数组
  val array = Array.ofDim[Int](2,3)
  
  //5.创建指定范围的数组
  val array = Array.range(5,1,-1) //5,4,3,2
  
  //6.创建具有初始化值的数组
  val array = Array.fill[Int](5)(1) //1,1,1,1,1
  ```
  
- 操作语法（对不可变数组的操作需要使用另外的数组对象接收）

  ```scala
  val array = Array(1,2,3,4)
  
  //1.添加数据
  val newArray = array :+ 5 //添加数据到数组末尾
  val newArray = 5 +: array //添加数据到数组开头
  
  //2.合并数组或集合
  val newArray = array ++ List(7,8,9) //合并数组或集合
  val newArray = Array.concat(array,array,array) //合并数组
  
  //3.遍历数组,执行自定义函数
  array.foreach(println)
  ```

#### 7.2.2 可变Array

- 扩展语法（不可变数组的所有操作方式在可变数组中都可以使用）

  ```scala
  //1.创建数组
  val arrayBuffer = new arrayBuffer[Int]()
  val arrayBuffer = ArrayBuffer(1,2,3,4)
  
  //2.添加数据
  arrayBuffer.append(5,6,7) //在数组末尾追加数据
  arrayBuffer.insert(2,5,6,7) //在指定索引位置插入追加的数据
  
  //3.修改数据
  arrayBuffer.update(0,11) //修改指定索引位置的数据,索引不能越界
  
  //4.删除数据
  arrayBuffer.remove(2) //删除指定索引位置的数据
  arrayBuffer.remove(2,2) //删除指定索引位置起的多个数据
  ```

#### 7.2.3 Array的转换

- Array和ArrayBuffer之间支持相互转换

  ```scala
  // 不可变=>可变
  val arrayBuffer = array.toBuffer
  // 可变=>不可变
  val array = arrayBuffer.toArray
  ```



### 7.3 Seq序列

- List是Seq集合中的常用实现类，类似于Java中的ArrayList，存储有序的、可重复的数据
- List是抽象类，无法直接构建对象，需要使用伴生对象的apply方法创建
- 可变与不可变List
  - 不可变List（List）：索引不可变，对集合元素的修改需要使用新的集合接收
  - 可变List（ListBuffer）：集合元素可修改，长度可伸缩

#### 7.3.1 不可变List

```scala
//1.创建集合
val list = List(1,2,3,4)

//2.遍历集合
println(list) //集合对象可直接遍历输出 List(1,2,3,4)
println(list.mkString(",")) //使用mkstring遍历
list.foreach(println) //使用自定义方法遍历集合

//3.添加数据(使用新集合接收)
val newList = list :+ 5 //添加数据到集合末尾
val newList = 5 +: list //添加数据到集合开头

//4.使用空集Nil创建集合
val nil = Nil //Nil等价于List()
val newList = list :: 5 :: 6 :: Nil //使用Nil连接生成集合,新集合List(1,2,3,4),5,6
val newList = list ::: 5 :: 6 :: Nil //使用Nil连接生成集合,并扁平化处理list,新集合1,2,3,4,5,6

//5.连接多个集合
val newList = List.concat(list,list)

//6.创建具有默认值的集合
val list = List.fill[String](5)(1)
```

#### 7.3.2 可变List

```scala
//不可变List中的操作依然可用
//1.创建集合
val listBuffer = ListBuffer(1,2,3,4)

//2.添加数据
listBuffer.append(5,6) //集合末尾添加数据
listBuffer.insert(0,5,6) //指定索引处插入数据

//3.修改数据
listBuffer.update(0,11) //修改指定索引位置的数据

//4.删除数据
listBuffer.remove(0) //删除指定索引位置的数据
listBuffer.remove(0,2) //删除指定索引位置起多个数据
```

#### 7.3.3 List的转换

```scala
// 不可变 => 可变
val listBuffer = list.toBuffer
// 可变 => 不可变
val list = listBuffer.toList
```



### 7.4 Set集

- Scala中的Set集合和Java中的Set集合类似，存储无序的、不可重复的数据
- 可变与不可变Set
  - 不可变Set（scala.collection.immutable）：索引不可变，修改集合元素时需要使用新集合接收
  - 可变Set（scala.collection.mutable）：集合元素可修改，长度可伸缩

#### 7.4.1 不可变Set

```scala
//1.创建集合
val set = Set(1,2,3,4)

//2.遍历集合
println(set) //直接遍历输出集合
println(set.mkstring(",")) //使用mkstring遍历输出集合
set.foreach(println) //使用自定义方法输出集合

//3.添加和删除数据
val newSet = set + 5
val newSet = set - 2

//4.合并集合
val newSet = set ++ set

//5.集合的交集与差集
val set = set1 & set2  //求set1和set2共有的元素的集合
val set = set1 &~ set2 //求set1有,set2没有的元素的集合
```

#### 7.4.2 可变Set

```scala
//1.创建集合
val set = mutable.Set(1,2,3,4)

//2.添加数据
set += 5
set.add(5)

//2.修改数据
set.update(3,true)  //更新后的集合中含有3
set.update(3,false) //更新后的集合中不含3

//3.删除数据
set -= 2
set.remove(2)
```



### 7.5 Map映射

- 和Java中的Map集合类似，Scala中的Map集合同样用于存储key-value结构的数据
- 存储时无序的，key和value可以为空，key不能重复
- 可变与不可变Map
  - 不可变Map（immutable.Map）：集合索引不可修改，修改元素时需要使用新的集合接收
  - 可变Map（mutable.Map）：集合元素可修改，长度可伸缩

#### 7.5.1 不可变Map

```scala
//1.创建集合
val map = Map(1 -> "a", 2 -> "b", 3 -> "c") //使用返回键值对的方法创建
val map = Map((1,"a"), (2,"b"), (3,"c")) //使用元组创建

//2.遍历集合
println(map) //直接输出遍历(格式为键值对 1 -> aaa)
println(map.mkString(",")) //使用mkstring遍历(格式为键值对 1 -> aaa)
map.foreach(println) //使用自定义方法遍历(格式为元组 (1,aaa))

//3.添加和删除数据
val newMap = map + (4 -> "d") //添加键值对
val newMap = map - 1 //使用key删除

//4.合并集合
val map = map1 ++ map2

//5.使用key获取value
map.getOrElse(1,-1) //获取key为1的valu,若为空则返回-1

//6.获取keys,values
map.keys
map.values
```

#### 7.5.2 可变Map

```scala
//1.创建集合
val map = mutable.Map(1 -> "a", 2 -> "b", 3 -> "c")
val map = mutable.Map((1,"a"), (2,"b"), (3,"c"))

//2.添加数据
map.put(4,"d")

//3.修改数据
map.put(5,"e") //若key不存在则视为添加数据

//4.删除数据
map.remove(3) //使用key删除数据
```

#### 7.5.3 Map的转换

```scala
//Map转换为其他集合(键值对转换为元组)
map.toSet
map.toList
map.toArray
```



### 7.6 Tuple元组

- 概述

  1. 在Scala中，可以将相互无关的任意数据类型的多个数据使用（）封装为1个整体，这个整体称为元组
  2. 元组的数据类型为（数据类型1，数据类型2...）
  3. 1个元组中最多存放22个元素
  4. 只有2个元素的元组称为对偶元组（键值对）

- 操作语法

  ```scala
  //1.元组的创建
  val data = (1,"atguigu",3000)
  
  //2.元组的调用
  val a = data._1 //使用顺序号调用元组元素(从1开始)
  val a = data.productElement(0) //使用索引调用元组元素(从0开始)
  
  //3.遍历元组
  println(data) //直接输出遍历元组
  val iterator = data.productIterator //使用迭代器遍历元组
  while (iterator.hasNext){
      println(iterator.next())
  }
  ```



### 7.7 常用方法

#### 7.1.1 操作集合

```scala
val list = List(1,2,3,4,5)

//1.集合长度(元素数量)
list.size
list.length

//2.判断集合是否为空
list.isEmpty

//3.迭代器
list.iterator

//4.判断是否包含元素
list.contains(1)

//5.取前n个元素
list.take(3)

//6.取后n个元素
list.takeRight(3) //输出顺序为3,4,5

//7.丢弃前n个元素
list.drop(3)

//8.丢弃后n个元素
list.dropRight(3)

//9.反转
list.reverse

//10.去重
list.distinct

//111.最大值
list.max

//12.最小值
list.min

//13.求和
list.sum

//14.求积
list.product
```



#### 7.1.2 衍生集合

```scala
val list = List(1,2,3,4,5)

//1.取首尾相关的元素
list.head //取第一个元素
list.last //取最后一个元素
list.init //取除最后一个元素之外的所有元素
list.tail //取除第一个元素之外的所有元素

//2.两集合操作
list1.union(list2) //并集
list1.intersect(list2) //交集
list1.diff(list2) //差集

//3.切分集合
list.splitAt(2) //得到两个集合组成的集合,第一个集合长度为2

//4.zip 拉链
//功能 将两个集合中的元素一对对组成元组作为新集合的一个元素,两集合长度不同时,多出的数据舍弃
list1.zip(list2)

//5.zipWithIndex 自关联
//功能 将集合中的每个元素和元素的索引组成一个元组作为新集合的一个元素(等同于和自身的索引做拉链)
list.zipWithIndex

//6.sliding 滑动
//功能 将集合中的元素按照指定的长度和步长切分为多个多个集合
list.sliding(3,1).toList //返回(1,2,3),(2,3,4),(4,5,6)
```



#### 7.1.3 功能函数

```scala
//1.map 映射转换
//功能 自定义函数,以集合中的每个元素为参数,返回任意类型的结果,对集合中的元素遍历转换
val list = List(1,2,3,4,5)
val newList = list.map(_ * 3)

//2.flatten 扁平化
//功能 将集合中的集合拆分一次,要集合中的集合类型必须一致
val list = List( List(1, 2), List(3, 4), List(5, 6), List(7, 8) )
val newList = list.flatten

//3.flatMap 映射+扁平化
//功能 自定义函数,以集合中的每个元素为参数,返回可迭代的集合类型的结果,对集合中的元素进行一次遍历转换后再进行一次扁平化(先map一次,再flatten一次)
val list = List( List(1, 2), List(3, 4), List(5, 6), List(7, 8) )
def fun(list : List[Int]) : List[Int] = {
    list.map(_ * 2)
}
val newList = list.flatMap(fun)

//4.filter 过滤
//功能 自定义函数,以集合中的每个元素为参数,返回Boolean型结果,保留结果为true的元素,丢弃结果为false的元素
val list = List("hello","scala","spark","hadoop")
val newList = list.filter(_.startWith("s"))

//5.groupBy 分组
//功能 自定义函数,以集合中的每个元素为参数,返回任意类型的结果,最终得到Map集合,函数的返回值作为Map集合的key,key值相同的元素组成的List集合作为Map集合的value
val list = List("hello","scala","spark","hadoop")
val map = list.groupBy(_.substring(0,1))

//6.sortBy 排序
//功能 自定义函数,以集合中的每个元素为参数,返回排序依据的数据,默认为数值降序和字典顺序
val list = List(1,2,3,4,5)
val newList = list.sortBy(num => num)(Ordering.Int.reverse) //(Ordering.Int.reverse)表示降序
//多个排序规则可使用元组封装并返回
val newList = list.sortBy(user => (user.name, user.age))(Ordering.Tuple2(Ordering.String, Ordering.Int.reverse))

//7.sortWith 排序
//功能 自定义函数,以集合中的两个元素为参数,返回Boolean型结果,返回true时,left会排在right前
list.sortWith(
	(left, right) => {
        left >= right
    }
)

//8.reduce 聚合运算 (A1, A1 => A1)
//功能 自定义函数,以集合中的两个元素为参数,返回和参数类型一致的结果,两两迭代运算整个集合
val list = List(1,2,3,4)
val newList = list.reduce(_+_)
//reduceLeft 左聚合 (B, A1) => B
//同样以集合中的两个元素为参数,但返回值类型始终与集合中的第一个参数一致,两两迭代运算整个集合 
val newList = list.reduceLeft(_+_)
//reduceRight 右聚合 (A1, B) => B
//从集合结尾向前两两迭代运算,返回值类型始终与集合最后一个参数一致
val newList = list.reduceRight(_-_) //运算顺序为 1-(2-(3-4))

//9.fold 折叠 (A1)(A1, A1 => A1)
//功能 自定义函数,以1个和集合元素类型相同的自定义变量和集合的1个元素作为参数,返回和参数类型一致的结果,两两迭代运算整个集合
val list = List(1,2,3,4)
val newList = list.fold(10)(_+_) //20
//foldLeft 左折叠 (B)(B, A1 => B)
//自定义变量可以是任意类型,每次迭代的返回值类型与自定义变量一致
val newList = list.foldleft("0")(_+_) //01234
//foldRight 右折叠 (B)(A1, B => B)
val newList = list.foldRight("0")(_+_) //12340
//foldLeft还可以用于合并Map
val map1: mutable.Map[String, Int] = mutable.Map("a"->1, "b"->2, "c"->3)
val map2: mutable.Map[String, Int] = mutable.Map("a"->4, "d"->5, "c"->6)
val newMap = map1.foldLeft(map2)(
    (map, kv) => {
        map.update(kv._1, map.getOrElse(kv._1, 0) + kv._2)
        map
    }
)

//10.scan 扫描
//功能 执行过程与fold相同,不同的是fold返回聚合后的结果,scan返回每一步两两运算结果组成的集合
val list = List(1,2,3,4)
val newList = scan(10)(_+_)
//scanLeft scanRight
//类似于左右折叠,输出每一步结果组成的集合
```



### 7.8 WordCount实现

#### 7.8.1 示例1

```scala
//读取数据文件,统计WordCount
hello scala
hello java
hadoop maven JDBC
scala flume flink
spark kafka
hive zookeeper
linux
shell linux
azkaban sql mysql
java
hive hadoop word spark linux
```

```scala
object WordCount {
    def main(args: Array[String]): Unit = {

        //1.读取数据文件
        val lines: List[String] = Source.fromFile("input/inputtest").getLines().toList

        //2.拆分为单词
        val list: List[String] = lines.flatMap(_.split(" "))

        //4.分组
        val map: Map[String, List[String]] = list.groupBy(word => word)

        //5.统计单词频率
        val countMap: Map[String, Int] = map.mapValues(_.length)

        //6.排序
        val countList: List[(String, Int)] = countMap.toList
        val sortedList: List[(String, Int)] = countList.sortBy(_._2)(Ordering.Int.reverse)

        //7.输出前3
        println(sortedList.take(3))
    }
}
```



#### 7.8.2 示例2

```scala
object WordCountExer01 {
    def main(args: Array[String]): Unit = {

        //方式1:将集合中的元组彻底拆分为单个单词,再分组求和
        val list = List(
            ("hello", 4),
            ("hello spark", 3),
            ("hello spark scala", 2),
            ("hello spark scala hive", 1)
        )

        //1.集合中每个元组转换为长字符串
        val lines: List[String] = list.map(
            (kv: (String, Int)) => {
                (kv._1 + " ") * kv._2
            }
        )

        //2.集合中每个长字符串通过切割扁平化处理为独立单词的集合
        val words: List[String] = lines.flatMap(_.split(" "))

        //3.将集合中元素按单词分组
        val map: Map[String, List[String]] = words.groupBy(word => word)

        //4.计算每个单词的出现次数
        val countMap: Map[String, Int] = map.mapValues(_.length)

        //5.转换为List集合后按照数量降序排序
        val countList: List[(String, Int)] = countMap.toList
        val sortedList: List[(String, Int)] = countList.sortBy(_._2)(Ordering.Int.reverse)

        //6.取数量前三的单词
        val result: List[(String, Int)] = sortedList.take(3)

        //7.输出结果
        println(result)
    }
}
```



#### 7.8.3 示例3

```scala
object Test02 {
    def main(args: Array[String]): Unit = {

        //统计不同省份的商品点击排行
        val list = List(
            ("zhangsan", "河北", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "鞋"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河南", "衣服"),
            ("wangwu", "河南", "鞋"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "鞋"),
            ("zhangsan", "河北", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "帽子"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河南", "衣服"),
            ("wangwu", "河南", "帽子"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "帽子"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "电脑"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河南", "衣服"),
            ("wangwu", "河南", "电脑"),
            ("zhangsan", "河南", "电脑"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "帽子")
        )

        //1.清洗姓名字段
        val etlList: List[(String, String)] = list.map( tuple => (tuple._2, tuple._3) )

        //2.按照省份分组
        val provMap: Map[String, List[(String, String)]] = etlList.groupBy( _._1 )

        //3.省份内对商品类型分组
        val commMap: Map[String, Map[String, List[(String, String)]]] = provMap.mapValues( _.groupBy( _._2) )

        //4.统计省份内商品的点击数
        val countMap: Map[String, Map[String, Int]] = commMap.mapValues( _.mapValues(_.length) )

        //5.修改集合类型可排序
        val countList: Map[String, List[(String, Int)]] = countMap.mapValues( _.toList )

        //6.省内排序
        val result: Map[String, List[(String, Int)]] = countList.mapValues( _.sortBy( _._2 )(Ordering.Int.reverse) )

        //7.输出结果
        println(result)

    }
}
```



### 7.9 Queue队列

- 队列：存储多个任意数据类型数据的数据结构，特点是先进先出

- 操作语法

  ```scala
  //1.创建队列
  val queue: mutable.Queue[String] = mutable.Queue[String]()
  
  //2.添加数据
  queue.enqueue("a")
  queue.enqueue("b","c","d")
  queue += "e"
  
  //3.删除数据(自动删除先加入的数据)
  queue.dequeue()
  ```



### 7.10 并行

- scala提供了并行集合，用于多核环境的并行计算

- 操作语法

  ```scala
  val result = (0 to 100).par.map{x => Thread.currentThread.getName}
  println(result)
  ```



## 第八章 模式匹配

### 8.1 基本语法

-  概述

  1. Scala中的模式匹配与Java中的swiich-case分支语法类似，但功能更加强大
  2. 与Java不同的是，**Scala中分支语句会在执行后退出分支结构**，即默认加入了Java中的break子句
  3.  若未匹配到case子句，程序运行会报错

- 语法示例

  ```scala
  val a = 10
  val b = 20
  val operate = '+'
  val result = operate match {
      case '+' => a + b
      case '-' => a - b
      case '*' => a * b
      case '/' => a / b
      case _ => "nothing"
  }
  println(result)
  //case _表示省略匹配的类型,即匹配所有operate,相当于Java中的default
  //若match的参数需要在case子句逻辑中使用,可以对参数重新命名并使用
  ```



### 8.2 扩展语法

#### 8.2.1 匹配常量

```scala
//定义函数,使用参数进行模式匹配
def matchtest(x : Any) = {
    val result = x match {
        case 5 => "Int 5"
        case "atguigu" => "String atguigu"
        case true => "Boolean true"
        case _ => "else"
    }
    result
}
```



#### 8.2.2 匹配数据类型

1. **数据类型的匹配不考虑泛型**
2. 底层逻辑使用类型判断实现

```scala
//定义函数,根据参数的类型进行模式匹配
def matchType(x: Any) = {
    val result = x match {
        case i : Int => "Int"
        case s : String => "String"
        case List[Int] => "List()"
        case Array[Int] => "Array[Int]"
        case other => "other"
    }
    result
}
//注意除数组外,集合的类型匹配不考虑泛型
println(matchType(List(1)))			//List()
println(matchType(List("atguigu")))	 //(List)
println(matchType(Array(1)))		//Array[Int]
println(matchType(Array("atguigu"))) //other
```



#### 8.2.3 匹配数组

```scala
//匹配数组中的具体数据
val arr = Array(0,2,3,"")
def matchtest(arr : Array[_]) = {
    val result = arr match {
        case Array(0) => "数组中只有元素0"
        case Array(x,y) => "数组中有2个元素"
        case Array (0, _*) => "数组中首个元素为0" // _*表示任意个任意元素
        case _ => "other Array"
    }
    result
}
println(matchtest(arr))
```



#### 8.2.4 匹配集合

```scala
//匹配集合中的元素
val list = List(0,1,2,"")
def matchList (list : List[_]) = {
    val result = list match {
        case List(0) => "集合中只有元素0"
        case List(x,y) => "集合中有2个元素"
        case List(0, _*) => "集合中首个元素为0"
        case _ => "other List"
    }
    result
}
println(matchList(list))
```



#### 8.2.5 匹配元组

```scala
//匹配元组中的数据
val tuple = (0,1)
val result = tuple match {
    case (0, _) => "第一个元素为0的对偶元组"
    case (x, 0) => "第二个元素为0的对偶元组"
    case (a, b) => "对偶元组"
    case _ => "else"
}
```



#### 8.2.6 匹配对象

```scala
object MatchObject {
    def main(args: Array[String]): Unit = {

        val user = User("atguigu",20)
        
        val result = user match{
            case User("atguigu",20) => "yes"
            case _ => "no"
        }
        println(result)

    }
}

class User(val name : String, val age : Int){}

object User{

    def apply(name: String, age: Int): User = new User(name, age)

    //scala中匹配对象会调用伴生对象的unapply方法
    def unapply(arg: User): Option[(String, Int)] = Some(arg.name, arg.age)

}
```



#### 8.2.7 样例类 case class

- 概述

  1. 使用case关键字声明的类即为样例类，用于方便的实现自定义类的模式匹配
  2. 样例类会自动生成伴生对象，并提供apply，unapplay，equals，hashCode，copy等常用方法
  3. 类的属性需要定义在主构造方法的参数列表中，便于case子句的使用
  4. 未显式声明时，主构造方法中的参数默认为val型

- 定义语法

  ```scala
  case class User(name : String, age : Int)
  
  object MatchObject {
      def main(args: Array[String]): Unit = {
  
          val user = User("atguigu",20)
  
          val result = user match{
              case User("atguigu",20) => "yes"
              case _ => "no"
          }
          println(result)
      }
  }
  ```



### 8.3 应用

#### 8.3.1 声明变量

```scala
val Array(first, second, _*) = Array(1,2,3,4)
println(first) //1
//等价于声明了2个变量first,second,分别赋值为1,2

case class Person(name:String,age:Int)
val Person(name,age) = Person("zhangsan",20)
println(name) // zhangsan
//等价于声明了个变量name,age,分别赋值为zhangsan,20
```



#### 8.3.2 循环匹配

```scala
//输出map集合中value为0的key值
val map = Map("a"->1,"b"->0,"c"->3)
for ( (k,0) <- map ) {
    println(k)
}

//等价于以下语法
for ( kv <- map ) {
    if (kv._2 == 0){
        println(kv._1)
    }
}
```



#### 8.3.2 函数参数

```scala
//将list中每个元素的第二个元素*2后返回
val list = List( ("a",1), ("b",2), ("c",3) )

val list1: List[(String, Int)] = list.map(
    kv => (kv._1, kv._2 * 2)
)

//模式匹配用于参数的解析,case匹配传入的参数
//模式匹配必须用{}
val list2 : List[(String, Int)] = list.map {
    case (word, count) => {
        (word, count*2)
    }
}
```



### 8.4 偏函数 PartialFunction

- 定义：**对集合中满足指定条件的数据进行处理，返回所有处理结果组成的新集合**

- 功能类似于map，区别在于使用偏函数时，若集合中的元素不满足所有分支条件，执行时不会报错，该元素直接丢弃

- 基本语法

  ```scala
  val list = List(1, 2, "3", 4)
  
  //定义偏函数
  val pf: PartialFunction[Any, Any] = {
      case i: Int => i * 2
      case s => s
  }
  
  val list1: List[Any] = list.collect(pf)
  ```
  
  
  
- 示例

  ```scala
  //对集合中所有Int型数据*2,并去除字符串
  val list = List(1, 2, 3, "4")
  list.collect{
      case i : Int => i * 2
  }
  ```



## 第九章 异常

### 9.1 概述

1. Scala中的异常与Java中大体相似，异常类型也直接使用类Java中的异常类
2. Scala中使用try-catch抓取异常时，多个异常间使用case子句区分
3. 多个case子句的抓取顺序与声明顺序一致，因此和Java一样，范围大的异常要后置声明
4. scala中没有throws关键字，因此无法显式抛出方法的异常

### 9.2 基本语法

- try-catch抓取异常

  ```scala
  try {
      val i = 0
      val j = 10 / i
  }catch {
      case e:ArithmeticException => {
          println("ArithmeticException")
      }
      case e:Exception =>{
          e.printStackTrace()
      }
  }finally {
      println("finally")
  }
  ```

- throws[]

  ```scala
  //虽然scala中没有throws关键字,但提供了替代的标签
  object Dept{
      //throws[]与throws关键字作用完全一致,[]可声明异常的类型,同时支持多次声明
      @throws[ArithmeticException]
      @throws[Exception]
      def test(): Unit ={
          throw new ArithmeticException
      }
  }
  ```



## 第十章 隐式转换

### 10.1 概述

- 在Scala中，没有数据精度的概念，因此 Int 型变量和 Double 型变量本质上是不能直接转换的，两个不同数据类型数据的自动转换必须具有子父类关系。

- 在Scala.Predef类中，默认提供了一系列的数据转换方法，使得如下代码可以正确的执行.

  ```scala
  val i: Int = 100
  val d: Double = i //Int型变量通过隐式转换方法转换为Double类型变量
  ```

- 同样的，我们可以自定义隐式转换方法，提供转换的规则，实现任意的数据类型之间的转换。**不存在关系的两个数据类型的数据在转换时，编译器会自动寻找转换的方法并尝试转换，这一过程就称为隐式转换**。

- 编译器尝试隐式转换时的自动检索过程

  1. 在当前代码作用域检索
  2. 在上级代码作用域检索
  3. 在类的包对象中检索
  4. 在类的父类、特质中检索
  5. **为避免自动检索失败，可直接使用 import 导入**



### 10.2 隐式函数

- 使用 implicit 关键字定义隐式函数

- **在同一作用域下不能定义两个参数类型和返回值类型完全相同的隐式函数**

- 自定义用于类型转换的隐式函数

  ```scala
  //从Double到Int的类型转换默认编译错误
  val i : Int = 2.0
  
  //自定义隐式函数,实现隐式转换
  implicit def transform (d: Double): Int = {
      d.toInt
  }
  ```

- 使用隐式函数实现功能的扩充

  ```scala
  //需求:在不改变源代码的基础上,对Person类的功能进行扩充
  class Person {}
  
  object ImplicitTest {   
      def main(args: Array[String]): Unit = {                
  
          //1.将扩充的方法使用另外的的自定义类封装
          class Extra {
              def fun() = { println("ExtraFunction") }
          }
          
          //2.声明隐式函数,以Person类对象作为参数,返回自定义类对象
          implicit def transform(p: Person): Extra = {
              new Extra
          }
          
          //3.使用Person类对象调用扩充的方法
          val p = new Person
          p.fun()
          //编译阶段的过程: 1.首次编译出错 2.编译器寻找隐式转换方法并调用 3.再次编译通过
      
      }      
  }
  ```



### 10.3 隐式参数和隐式变量

- 概述

  1. 有些函数的参数需要定义默认值，而默认值在后续的开发中可能需要更改，这时可以使用隐式参数声明，并使用隐式变量赋值。
  2. 隐式参数：要求**参数列表有且仅有一个参数**（可使用函数柯里化实现），**且必须在调用函数时省略()**
  3. 隐式变量：1个隐式变量可以多次赋值为多种数据类型，但1种数据类型只能赋值一次，隐式参数在使用时会自动寻找同名的隐式变量并使用匹配的数据类型赋值。

- 语法格式

  ```scala
  //定义含有隐式参数的函数
  def regist(id: String)(implicit password: String = "000000") = {
      println(id + " " + password)
  }
  
  //定义隐式变量并赋值
  implicit val password = "123123"
  implicit val password = 123456 //同名隐式变量可多次赋值为不同的数据类型
  
  //调用方法
  regist("atguigu")            //输出 atguigu 123123
  regist("atguigu")()          //输出 atguigu 000000
  regist("atguigu")("123456")  //输出atguigu 123456
  ```



### 10.4 隐式类

- 概述

  1. 隐式类是scala2.10起提供的新特性，可以更加方便的实现对类的功能的拓展
  2. 隐式类**必须定义在类、伴生对象、包对象中**
  3. 隐式类的**构造方法有且仅能有1个参数**

- 语法格式

  ```scala
  //需求；扩充person类的功能
  class Person {}
  
  object ImplicitTest {   
      def main(args: Array[String]): Unit = {                
  
          //1.自定义隐式类,参数列表使用需要转换的参数类型,定义扩充的方法
          implicit class Extra(p: Person) {
              def fun() = { println("ExtraFunction") }
          }
          
          //2.使用Person类对象直接调用扩充的方法
          val p = new Person
          p.fun()
          //编译阶段的过程: 1.首次编译出错 2.编译器寻找隐式类中的方法并调用 3.再次编译通过
      
      }      
  }
  ```





## 第十一章 泛型

### 11.1 概述

- Scala中的泛型与Java中泛型的含义相同，并在功能上进行了扩展
- Scala中泛型的声明使用 [ ]



### 11.2 泛型转换

- Scala中泛型默认不可变，即类型相同而泛型参数不同的类的对象之间不能相互转换
- 使用泛型协变和泛型逆变可以实现具有子父类关系的泛型参数的类的对象的转换

```scala
//定义具有子父类关系的类
class SuperClass{}
class TestClass extends superClass{}
class SubClass extends testClass{}

//定义三个不同的泛型类
class A[T] {}
class B[+T] {}
class C[-T] {}

object Generic {
    def main(args: Array[String]): Unit = {
        
        //1.泛型不可变,泛型与类型的整体作为对象的类型
        val a1: A[TestClass] = new A[Superclass] //error
        val a2: A[TestClass] = new A[TestClass]
        val a3: A[TestClass] = new A[SubClass] //error
        
        //2.泛型协变:使用+T声明泛型,使子类泛型对象可以向父类泛型对象转化
        val b1: B[TestClass] = new B[Superclass] //error
        val b2: B[TestClass] = new B[TestClass]
        val b3: B[TestClass] = new B[SubClass]
        
        //3.泛型逆变:使用-T声明泛型,使父类泛型对象可以向子类泛型对象转化
        val c1: C[TestClass] = new C[Superclass]
        val c2: C[TestClass] = new C[TestClass]
        val c3: C[TestClass] = new C[SubClass] //error
        
    }
}
```



### 11.3 泛型边界

- 在scala中，可以使用泛型边界设定泛型参数的范围

- 泛型上限

  ```scala
  //定义泛型方法并声明泛型上限
  def test[T <: Person](t : T) = {
      println(t)
  }
  //说明:调用方法时,泛型参数必须为Person类或其子类
  test[Person](new Person())
  test[SubPerson](new SubPerson())
  ```

- 泛型下限

  ```scala
  //定义泛型方法并声明泛型下限
  def test[T >: Person](t : T) = {
      println(t)
  }
  //说明:调用方法时,泛型参数必须为Person类或其父类
  test[Person](new Person())
  test[SuperPerson](new SuperPerson())
  ```



## 第十二章 正则表达式

### 12.1 基本语法

```scala
//1.声明规则
val r1: Regex = "^\\d{11}$".r

//2.使用规则
val p1 = "18812341234"
val p2 = "falkflkgla"
val maybeString: Option[String] = r1.findFirstIn(p1)
if (!maybeString.isEmpty){
    println(maybeString.get)
}
```



### 12.2 常用校验

- 校验手机号

  ```scala
  def isMobileNumber(number: String): Boolean = {
      val regex = "^((13[0-9])|(14[5,7,9])|(15[^4])|(18[0-9])|(17[0,1,3,5,6,7,8]))[0-9]{8}$".r
      val length = number.length
      regex.findFirstMatchIn(number.slice(length-11,length)) != None
  }
  ```

- 提取邮箱域名

  ```scala
  val r = """([_A-Za-z0-9-]+(?:\.[_A-Za-z0-9-\+]+)*)(@[A-Za-z0-9-]+(?:\.[A-Za-z0-9-]+)*(?:\.[A-Za-z]{2,})) ?""".r
  println(r.replaceAllIn("abc.edf+jianli@gmail.com   hello@gmail.com.cn", (m => "*****" + m.group(2))))
  ```





## 第十三章 Scala实战

### 13.1 编写分布式计算框架

