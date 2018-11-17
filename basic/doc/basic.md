# 基础语法

## DOS常用命令

| 命令  | 参数            | 作用                                   |
|-------|-----------------|----------------------------------------|
| dir   | &nbsp;          | 查看当前目录下所有信息                 |
| cd    | [路径]          | 进入指定目录                           |
| cd    | ..              | 返回上一级目录                         |
| cd    | \\              | &nbsp;                                 |
| cd    | /               | 返回当前盘符根目录                     |
| md    | [目录名]        | 新建目录                               |
| del   | [文件名.扩展名] | 删除文件（多文件删除时空格隔开文件名） |
| del   | [目录名]        | 清空当前目录                           |
| rmdir | [目录名]        | 删除目录                               |
| exit  | &nbsp;          | 退出cmd                                |

## 关键字

> 数字、字母、特殊字符（_、$），不可使用关键字、开头不能使用数字

## 标识符

### 命名规范

| 类别      | 规范                                   |
| ------- | ------------------------------------ |
| 包名      | 单级包全小写                               |
| &nbsp;  | 多级包全小写，用`.`分隔                        |
| 方法名、变量名 | 单词首字母小写后续首字母大写                       |
| 常量名     | 单词全大写，下划线分隔字母                        |
| &nbsp;  | &nbsp;                               |
| 编译包名    | javac [-d] [.class文件所在路径] [.class文件] |
| 运行包名    | java [-cp]xx/xx/xx.java              |

## 注释

**单行注释**

以 // 开头后一行内容不被编译

**多行注释**

```java
/*
 *  不可嵌套多行注释
 */
```
 
**文档注释（javadoc）**

```java
/**
 *  @xxxx
 */
```

## 进制

| 进制   | 范围      | 标识     |
| ---- | ------- | ------ |
| 二进制  | 0-1     | 0b     |
| 八进制  | 0-7     | 0      |
| 十进制  | 0-9     | &nbsp; |
| 十六进制 | 0-9,A-F | 0x     |

> 运算的数据底层都是根据二进制数据的补码

* 原码：正数符号位为0，负数为1
* 反码：正数反码和原码相同，负数反码符号位不变，数值位按位取反
* 补码：正数补码和原码相同，负数补码 = 反码 + 1
 
## 常量

**字面值常量**

* 字符串常量
    * "内容"
* 字符常量
    * 'test'
* 整数常量
    * 123
* 小数常量
    * 1.1
* 布尔常量
    * false，true
* 空常量
    * null

## 数据类型

> 变量的初始化**小数**默认`double`，**整数**默认`int`；`long--L`；`float--F`

### 基本数据类型

| 类型      | 字节     | 范围                         |
| ------- | ------ | -------------------------- |
| **整型**  | &nbsp; | &nbsp;                     |
| byte    | 1      | $-2^7$~$2^7$ $-$ $1$       |
| short   | 2      | $-2^{15}$~$2^{15}$ $-$ $1$ |
| int     | 4      | $-2^{31}$~$2^{31}$ $-$ $1$ |
| long    | 8      | $-2^{63}$~$2^{63}$ $-$ $1$ |
| **浮点数** | &nbsp; | &nbsp;                     |
| float   | 4      | $-3.04E38$~$3.04E38$       |
| double  | 8      | $-1.798E308$~$1.798E308$   |
| **字符**  | &nbsp; | &nbsp;                     |
| char    | 2      | $0$~$65535$                |
| **布尔**  | &nbsp; | &nbsp;                     |
| boolean | 1      | &nbsp;                     |

### 引用类型

1. 数组
2. 接口
3. 类

## 数据类型转换

### 默认转换

由小到大（由高到低会损失精度）
底层默认int类型，使用时进行强制转换

``` java		
char/short/byte ----> int ----> long ----> float ----> double
```

### 强制转换
		
``` java
[类型] 变量名 = [目标类型] 数值;
```

## 运算符

``` java
()->!->算术->关系->逻辑->赋值
```

* `java`中，允许小数除以`0`，会得到`infinity`（无穷大）
* `+=`等**赋值运算符**在运算时自动进行**强制类型转换**
* **关系运算符**结果一定为`boolean`：`true`或`false`
    * `==`只能用于**基本数据类型**（4类8种）
* **逻辑运算符**用于连接布尔表达式或者值
    * `&`：全真为真
    * `&&`：**短路逻辑** ---> 条件a&&条件b ---> 如果左边为真，执行右边（继续判断）；**如果左边为假，不执行右边**（停止判断）
	* `|`：有真则真
	* `||`：**短路逻辑** ---> 条件a||条件b ---> **如果左边为真，不执行右边**（停止判断）；如果左边为假，执行右边（继续判断）
	* `!`：取反
	* `^`：**同假异真**
* **位运算符**运算时转换成二进制形式的**补码**计算
    * `&`（按位与）：遇0则0
	* `|`（按位或）：遇1则1
	* `^`（按位异或）：`同0异1`，异或多次时，偶数次为本身
	* `~`（按位取反）：所有位按位取反 ---> 补码 ---> 减1 ---> **符号位不变，其余位取反**
	* `>>`（右移）
		**正数**右移，最高位**补`0`**
		**负数**右移，最高位**补`1`**
		**被操作数除以`2`的移动位数次幂**
	* `<<`（左移）
		**左移**，最低位**补`0`**
		**被操作数乘以`2`的移动位数次幂**
	* `>>>`（无符号右移）
		**右移**，最高位**补`0`**

## 循环控制

* `switch`中，`case`缺失`break`造成**穿透效应**
* `switch`条件可以使用`String`--`jdk1.7`

## 数组

* 数组长度属性为`arr.length`

| 类型             | 初始值      |
| -------------- | -------- |
| int、short、byte | 0        |
| long           | 0L       |
| double         | 0.0      |
| float          | 0.0F     |
| char           | '/u0000' |
| 布尔类型           | false    |
| 引用类型           | null     |

## 内存

使用`new`实例化对象，在**堆**上分配内存

> **引用类型**赋值传递的是**地址**，**基本类型**赋值传递的是**值**

* 栈：存储变量；内存不进行赋值操作，**使用后被清除**
* 堆：存储对象；内存会进行赋值操作，使用后不会被立即清除，特定时间被清除（**gc回收**）
* 方法区：内含核心类库
    * 通过类加载器寻找`.class`文件（`classloader`）
	* 被`static`标识的**变量**、**方法**等存放于**静态区**，并**赋初始值**
* 本地方法栈：（混合编程）
* 寄存器：（汇编）
* `for-each`：操作在**栈**上，获得堆内的值后被清除，**不改变堆的值**

### 数据扩容

> 创建新长度的数组，复制源数组，修改末尾元素值

``` java	
System.arraycopy(源数组，源数组起始位置，新数组，新数组起始位置);
Arrays.copyOf(源数组，目标大小);
```

### 数据定义

``` java
int [] y, x []; ---> int [] y, int [] x [];
```
* 如果[]跟在数据类型的后面，那么这个[]属于所有变量，每个变量最起码是一维数组；
* 如果[]跟在变量名的后面，那么这个[]只属于这个变量，只代表这个变量是数组。

## 方法

### 方法重载

* 发生在同类中
* 方法名相同，参数列表不同
    * 方法名(参数列表)---->方法签名
* 与**返回类型**和**访问权限**均无关

### 方法重写

* 发生在父子类
* 子类的**方法签名**和父类的**一致**，方法重写后，执行子类方法
* 方法名、方法签名、方法返回值类型（**基本类型**、`void`）一致
* 如果父类方法返回值类型是**引用类型**，那么子类的返回类型一定要和父类的**一致**或者是其**子类**
* 子类的**访问修饰符大于等于**父类的访问修饰符
* 子类异常类型不大于父类
    * 例如：父类`throwsIOException`，子类就必须`throwsIOException`或者`IOException`的子类

### 方法传参

* 如果传的是**基本数据类型**，传入的是**值**，对值进行任何操作都不会影响源数据
* 如果传的是**引用类型**，传入的是**地址**，对值进行任何操作都会**对地址指向的内存造成影响**

### 构造方法

> 无static，无返回值，方法名与类名一致

``` java
权限 构造名() {
    ...
}
```
* 在java中，所有的非静态方法和属性都能通过对象调用
* 构造方法**可以重载**
* 构造方法**不可以重写**

### this

> 代表的是当前类的对象

如果类还在定义的过程中，没有对象的时候，`this`可以代替对象调用当前类的属性和方法

#### this语句

> 调用本类其他的构造方法，放在首行

``` java
this(参数列表);
```

### super

> 代表父类对象的引用

* 可以调用父类的属性和方法
* 调用父类中相应形式的构造方法
* 父类对象创建一定在子类之前

#### super语句

> super语句一定放在首行，与this语句不能共存

``` java
super(参数列表);
```

## 代码块

* 构造代码块**优先于**任何形式的构造方法，**在类中定义**
* 局部代码块**定义在方法中**，可以控制变量的生命周期，提高开发效率

| &nbsp; | 定义位置   | 使用范围           | 生命周期                    |
| ------ | ------ | -------------- | ----------------------- |
| 成员变量   | 类      | 整个类，放在堆里       | 随着类的定义而产生，随着对象的释放而消失    |
| 局部变量   | 方法、代码块 | 所在的方法/代码块，放在栈里 | 随着方法的调用而产生，随着方法的执行完成的消失 |

## 访问修饰符

| &nbsp;同类  | 子类 | 同包类 | 其他类 |    |
| --------- | -- | --- | --- | -- |
| public    | 能  | 能   | 能   | 能  |
| protected | 能  | 能   | 能   | 不能 |
| 默认        | 能  | 同包  | 能   | 不能 |
| private   | 能  | 不能  | 不能  | 不能 |

## 多态

> 多态是用于提高代码的**灵活性**，**解耦**（高内聚，低耦合）

* 编译时多态：
    * 方法重载
* 运行时多态：运行时多态的前提是继承
    * 方法重写
    * 向上造型：
        * 父类声明对象，子类实现对象

## static

### 静态变量

在类加载到**方法区**的**静态常量池**的时候，静态变量随着类加载到**方法区**的**静态区**

* 静态变量优先于对象存在
* 静态变量可以通过类调用
* 多个对象引用指向同一个静态变量，对象**共享静态变量**
* 静态变量不能在**构造代码块**中定义，静态变量随着类的加载而加载，而构造代码块与对象同级
* 静态变量不能在**方法**中定义，方法需要对象调用，静态变量优先于对象存在

### 静态方法

在类加载到**方法区**的**静态常量池**，静态方法随着类加载到**静态区**，不做任何操作

* 当方法被调用时，会被加载到栈中执行
* 静态方法是优先于对象存在的
* 可以通过类或者对象调用
* 不可以在静态方法中定义静态变量，静态方法在调用时才能执行
* 不可以在静态方法中使用this，this代表当前类的对象引用，静态方法优先于对象存在
* 非静态方法不可以直接在静态方法中调用，非静态方法被对象调用，静态方法优先于对象存在
* 静态方法可以重载
* 静态方法可以被继承，不可以重写

> `java`中允许父子类中出现方法签名一致的**静态方法**，但**不可以重写**。父子类要么都是静态，要么都不是。

### 静态代码块

随着类的加载而加载，**仅加载一次**

### 静态内部类

静态内部类可以定义静态和非静态属性属性、构造方法、普通方法

* 仅能调用外部类的静态属性和方法

### 加载顺序

父类--->子类
	
父类静态->子类静态->父类非静态->子类非静态

* 有继承关系时，先加载父类
* 子类静态优先于父类非静态加载

> 静态变量的默认值先于赋值操作

## final

* 修饰的数据就是变量的值，常量。值不能发生改变
* 修饰数组时，修饰**地址值**（地址值是一个常量）
* 修饰基本数据类型时，传参的时候是基本数据的值的拷贝，如果是引用类型，是引用地址的拷贝。
* 修饰方法**可以重载**，但**不可以重写**
* 修饰的类**不可以被继承**，可以有父类，类里的方法不能重写

## abstract

* 抽象类中可以有构造方法，但不可以创建对象（底层由C构建对象）
* 继承抽象类的子类需要重写父类所有抽象方法，如果重写不完全，**子类需要变成抽象类**
* 构造方法可以私有化
* 抽象方法可以重载，也可以重写
* 不可以使用`final`和`static`
* 抽象类中不一定有抽象方法，但有抽象方法一定是抽象类

## 接口

> 所有的方法都是抽象方法

* 实现接口的类需要实现接口中所有抽象方法
* 接口可以多实现
* 接口之间可以多继承，形成继承网
* 接口中一定有抽象方法（`jdk1.7`及其以前，`1.8`后接口中可以定义其他成员或普通方法）
* 接口内**属性**默认修饰符：`（public）static final`
* 接口内**方法**默认修饰符：`（public）abstract`
* 编译时期为提高效率不检验继承关系，运行时期检验继承（实现）关系

## 内部类

> 类或者接口中定义类

### 方法内部类

> 方法内部类可以定义**属性**、**构造方法**、**普通方法**

* 方法内部类仅能在当前**方法内部创建对象**
* 方法内部类可以**调用外部类**所有属性和方法
* 方法内部类如果要调用本方法中的变量，需要将其变成常量（`final`）
    * （`jdk1.8`不需要`final` --- **隐式常量**，默认加载`final`；`jdk1.7`**显式常量**）
* 不可以`static`，可以`final`
* 属性可以`final` `static`

### 成员内部类

> 成员内部类可以定义属性、构造方法、普通方法

* 只能定义非静态属性和方法
* 可以调用外部类的属性和方法
* 可以使用所有访问权限修饰符

### 匿名内部类

> 匿名内部类相当于继承类，可以被继承的类或者接口都可以定义匿名内部类

* `{}`内容就是匿名内部类的内容
* `final`类中没有匿名内部类（`final`不能被继承）
* **构造方法私有化**，可以存在匿名内部类
* 如果匿名内部类在方法中，那么使用的时候和方法内部类一致

在`Java`中，类中可以定义**接口**或者**类**，接口中可以定义**类**或者**接口**

## 正则表达式

### 常用正则表达式

``` java
    /^[\u4e00-\u9fa5\w]+.*$/gi
```

> 文本输入（拒绝表情）

``` java
    /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,20}$/
```

> 密码

``` java
    /^([\u4e00-\u9fa5])+[\u4e00-\u9fa5a-zA-Z0-9()（）]*$/
```

> 中文地址（以中文开始，包含英文字符、数字、括号）

``` java
    /^\d{6}(18|19|20)?\d{2}(0[1-9]|1[12])(0[1-9]|[12]\d|3[01])\d{3}(\d|X)$/i
```

> 身份证

``` java
    /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
```

> 电子邮箱

``` java
    /^(\d{3,4}-)?\d{7,8}$/
```

> 传真

``` java
    /^([hH][tT]{2}[pP]:\/\/|[hH][tT]{2}[pP][sS]:\/\/)*(([A-Za-z0-9-~]+)\.)+([A-Za-z0-9-~\/])+$/
```

> 网址

``` java
    /^(\(\d{3,4}\)|\d{3,4}-|\s)?\d{7,14}$/
```

> 座机

``` java
    /^1[34578]\d{9}$/
```

> 手机

``` java
    /^[1-9][0-9]{5}$/
```

> 邮编

``` java
    /^\\d+\\.\\d+$/
```

> 小数

### 常见需要转义的正则表达式符号

| 正则 | 转义     |
|----|--------|
| .  | \\\\.  |
| +  | \\\\+  |
| ?  | \\\\?  |
| [] | \\\\[] |
| () | \\\\() |
| {} | \\\\{} |

* []中不能出现数量词
* 如果捕获组和替换部分不在一个表达式中，获取捕获内容使用`$`

## 包装类

为了使基本类型具备对象的特性，提供包装类

| 基本类型    | 包装类型      |
|---------|-----------|
| byte    | Byte      |
| short   | Short     |
| int     | Integer   |
| long    | Long      |
| float   | Float     |
| double  | Double    |
| char    | Character |
| boolean | Boolean   |

* 装箱/封箱：
    基本数据类型转换为对应的包装类对象
* 自动装箱/封箱：
    基本数据类型直接赋值给对应的包装类对象
    * `jdk1.5`之后
* 包装类对象与其对应基本类型变量进行运算时，会**自动拆箱**，把包装类转换为相应的基本类型
* 包装类对象通过指向堆内存来指向**常量池**
* **字面值**存放于**常量池**，挺高代码利用率
    * 字符串
    * 小数
    * 整数
    * 字符
    * 布尔
    * null

## 异常

> 顶级父类为`Throwable`，直接子类`Error`和`Exception`

* `Error`类以及它的子类的实例，代表`JVM`本身的错误，不能被程序员通过代码处理。
* `Exception`以及它的子类，代表程序运行时发送的各种不期望发生的事件。可以被`Java`异常处理机制使用， 是异常处理的核心。

### 非检查异常

`Error`和`RuntimeException`以及它们的子类，**运行时异常**。`javac`在编译时不会提示和发现这样的异常，不要求在程序中处理这些异常，可以不处理。这类异常应该修正代码，而不是通过异常处理器处理。

### 检查异常

除`Error`和`RuntimeException`的其他异常，**编译时异常**。`javac`强制要求为这样的异常做预备处理工作，要么使用`try-catch`语句捕获它并处理，要么用`throws`语句声明抛出，否则编译不通过。

### 自定义异常

如果要自定义异常类，则需要继承`Exception`类，这样的自定义异常属于**编译时异常**。如果要自定义**运行时异常**，需要继承`RuntimeException`类。

* 自定义的异常应该总是包含如下的**构造函数**
    * 一个**无参**构造函数
    * 一个带有`String`参数的构造函数，并传递给父类的构造函数
    * 一个带有`String`参数和`Throwable`参数，并都传递给父类构造函数
    * 一个带有`Throwable`参数的构造函数，并传递给父类的构造函数

``` java
public class IOEception extends Exception {
    static final long serialVersionUID = 7818375828146090155L;

    public IOEception() {
    }

    public IOEception(String message) {
        super(message);
    }

    public IOEception(String message, Throwable cause) {
        super(message, cause);
    }

    public IOEception(Throwable cause) {
        super(cause);
    }
}
```

### finally

> `finally`块不管异常是否发生，只要对应的`try`执行了，它就一定执行。

> 只有`System.exit()`可以让`finally`不执行。

* `finally`块通常用来做资源释放操作，如关闭文件、关闭数据库连接等
    * 在`try`块中打开资源，在`finally`块中清理释放这些资源
* `finally`块没有处理异常的能力
* 在一个`try-catch-finally`中，如果`try`抛出异常，且有匹配的`catch`块，先执行`catch`块再执行`finally`块，否则先执行`finally`块，然后去外面的调用者中寻找合适的`catch`块
* 在一个`try-catch-finally`中，如果`try`抛出异常，且匹配的`catch`块中处理异常时也抛出异常，那么后面的`finally`块也会执行：先执行`finally`块，然后去外面的调用者中寻找合适的`catch`块
* 在`try`块中即使有`return`、`break`、`continue`等改变执行流的语句，`finally`块也会执行
    * `try-catch-finally`中的`return`只要能执行，就都执行了，它们**共同向一个内存地址写入返回值，后执行的将覆盖先执行的数据**，而真正被调用者取的返回值就是**最后一次**写入的。
* `finally`中的`return`会**抑制**（消灭）前面`try`或者`catch`块中的异常
* `finally`中的异常会**覆盖**（消灭）前面`try`或者`catch`中的异常

### throw

> 可以通过`throw`语句手动显式地抛出一个异常。

* `throw`语句的后面必须是一个**异常对象**
* `throw`语句必须**写在函数中**
* 执行`throw`语句的地方就是一个异常抛出点，与`JRE`自动形成的异常抛出点没有任何差别

### 线程异常

`Java`程序可以是多线程的，每一个线程都是一个独立的执行流，独立的函数调用栈。

* 如果程序只有一个线程，那么没有被任何代码处理的异常会导致程序终止
* 如果程序是多线程，那么没有被任何代码处理的异常仅会导致异常所在的线程结束

> `Java`中的异常是**线程独立**的，线程的问题应该由线程自己来解决，而不是委托到外部，也不会直接影响到其它线程的执行。

## 集合

### Collection

接口是集合类的**根接口**，Java中没有提供这个接口的直接的实现类。但是却让其被继承产生了两个接口，就是`Set`和`List`。`Set`中**不能包含重复**的元素。`List`是一个**有序**的集合，**可以包含重复**的元素，提供了**按索引访问**的方式。

### List

`List`里存放的对象是**有序**的，同时也是**可以重复**的，`List`关注的是**索引**，拥有一系列和索引相关的方法，**查询速度快**。因为往`list`集合里**插入**或**删除**数据时，会伴随着后面数据的移动，所有插入删除数据速度慢。

#### ArrayList

`ArrayList`是一个**动态数组**，也是我们最常用的集合。它允许任何符合规则的元素插入甚至包括`null`。每一个`ArrayList`都有一个**初始容量**（`10`），该容量代表了数组的大小。**随着容器中的元素不断增加，容器的大小也会随着增加**。在每次向容器中增加元素的同时**都会进行容量检查**，**当快溢出时，就会进行扩容操作**。所以如果我们明确所插入元素的多少，最好指定一个初始容量值，避免过多的进行扩容操作而浪费时间、效率。
`size`、`isEmpty`、`ge`t、`set`、`iterator`和`listIterator`操作都以**固定时间**运行。`add`操作以**分摊**的固定时间运行，也就是说，添加`n`个元素需要`O(n)`时间。

* `ArrayList`擅长于随机访问。同时`ArrayList`是**非同步**的。

#### LinkedList

同样实现`List`接口的`LinkedList`与`ArrayList`不同，`ArrayList`是一个**动态数组**，而`LinkedList`是一个**双向链表**。所以它除了有`ArrayList`的基本操作方法外还额外提供了`get`，`remove`，`insert`方法在`LinkedList`的**首部**或**尾部**。
由于实现的方式不同，`LinkedList`**不能随机访问**，它所有的操作都是要按照**双重链表**的需要执行。在列表中索引的操作将从**开头**或**结尾**遍历列表（从靠近指定索引的一端）。这样做的好处就是可以通过较低的代价在`List`中进行插入和删除操作。
与`ArrayList`一样，`LinkedList`也是**非同步**的。如果多个线程同时访问一个`List`，则必须**自己实现访问同步**。一种解决方法是在创建`List`时构造一个同步的`List`：

``` java
    List list = Collections.synchronizedList(new LinkedList(...));
```

### Set

`Set`里存放的对象是**无序**，**不能重复**的，集合中的对象不按特定的方式排序，只是简单地把对象加入集合中。

#### HashSet

`HashSet`是一个**没有重复元素**的集合。它是由`HashMap`实现的，不保证元素的顺序(这里所说的没有顺序是指：**元素插入的顺序与输出的顺序不一致**)，而且`HashSet`允许使用`null`元素。`HashSet`是**非同步**的，如果多个线程同时访问一个`HashSet`，而其中至少一个线程修改了该`HashSet`，那么它必须保持**外部同步**。`HashSet`按`Hash`算法来存储集合的元素，因此具有很好的**存取**和**查找**性能。
`HashSet`通过元素的`hashcode()`和`equals()`方法进行判断元素是否重复。如果对象的`hashCode`值是不同的，那么`HashSet`会认为对象是不可能相等的，因此自定义类的时候需要**重写**`hashCode`，来确保对象具有相同的`hashCode`值。如果元素(对象)的`hashCode`值相同，会继续使用`equals`进行比较，如果`equals`为`true`那么新加入的对象重复了，所以加入失败。如果`equals`为`false`，那么`HashSet`认为新加入的对象没有重复，新元素可以存入。
`HashSet`的实现方式大致如下：通过一个`HashMap`存储元素，元素是存放在`HashMap`的`Key`中，而`Value`统一使用**一个**`Object`对象。

* `HashSet`中是允许存入`null`值的，但是在`HashSet`中仅仅能够存入**一个**`null`值。
* `HashSet`中存储的元素的是无序的，但是由于`HashSet`底层是基于`Hash`算法实现的，使用了`hashcode`，所以`HashSet`中**相应**的元素的位置是**固定**的。
* `HashSet`默认初始容量是`16`，加载因子是`0.75f `(`12`开始扩容，每次扩容**一倍**)
* 扩容后进行`rehash`，重新计算元素位置(内存地址%当前容量)

#### LinkedHashSet

`LinkedHashSet`继承自`HashSet`，其底层是基于`LinkedHashMap`来实现的，**有序**，**非同步**。`LinkedHashSet`集合同样是根据元素的`hashCode`值来决定元素的存储位置，但是它同时使用**链表**维护元素的次序。这样使得元素看起来像是以插入顺序保存的，也就是说，当遍历该集合时候，`LinkedHashSet`将会**以元素的添加顺序访问集合的元素**。

#### TreeSet

`TreeSet`是一个`有序`集合，其底层是基于`TreeMap`实现的，**非线程安全**。`TreeSet`可以确保集合元素**处于排序状态**。`TreeSet`支持两种排序方式，**自然排序**和**定制排序**，其中自然排序为默认的排序方式。当我们构造`TreeSet`时，若使用**不带参数**的**构造函数**，则`TreeSet`使用自然比较器；若用户需要使用**自定义的比较器**，则需要使用带比较器的参数。
`TreeSet`集合不是通过`hashcode`和`equals`函数来比较元素的.它是通过`compare`或者`comparaeTo`函数来判断元素是否相等.`compare`函数通过判断两个对象的`id`，相同的`id`判断为**重复元素**，**不会被加入到集合中**。

### Vector

与`ArrayList`相似，但是`Vector`是**同步**的。所以说`Vector`是**线程安全**的**动态数组**。它的操作与`ArrayList`几乎一样。

### Stack

`Stack`继承自`Vector`，实现一个**后进先出**的**堆栈**。`Stack`提供`5`个额外的方法使得`Vector`得以被当作堆栈使用。基本的`push`和`pop`方法，还有`peek`方法得到栈顶的元素，`empty`方法测试堆栈是否为空，`search`方法检测一个元素在堆栈中的位置。`Stack`刚创建后是**空栈**。

### Map

`Map`是`Java.util`包中的另一个接口，它和Collection接口**没有关系**，是**相互独立**的，但是**都属于集合类**的一部分。`Map`包含了`key-value`对。`Map`**不能包含重复**的`key`，但是**可以包含相同**的`value`。

> 对`map`集合遍历时先得到**键**的`set`集合，对`set`集合进行遍历，得到相应的值。

* `Map`的`key`**不允许重复**，即同一个`Map`对象的任何两个`key`通过`equals`方法比较总是返回`false`。
* `key`和`value`之间存在**单向一对一**关系，即通过指定的`key`,总能找到**唯一**的、**确定**的`value`。

#### HashMap

`HashMap`是最常用的`Map`，它根据键的`HashCode`值存储数据，根据键可以**直接获取**它的值，具有**很快的访问速度**，遍历时，取得数据的顺序是**完全随机**的。因为键对象**不可以重复**，所以`HashMap`**最多只允许一条记录的键为`Null`**，**允许多条记录的值为`Null`**，是非同步的。

#### Hashtable

`Hashtable`与`HashMap`类似，是`HashMap`的**线程安全**版，它支持线程的同步，即**任一时刻只有一个线程能写`Hashtable`**，因此也导致了`Hashtale`在**写入时**会比较**慢**，它继承自`Dictionary`类，不同的是它**不允许记录的键或者值为`null`**，同时**效率较低**。

#### ConcurrentHashMap

**线程安全**，并且锁分离。`ConcurrentHashMap`内部使用段`(Segment)`来表示这些不同的部分，每个段其实就是一个小的`hash table`，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以**并发**进行。

#### LinkedHashMap

`LinkedHashMap`保存了记录的**插入顺序**，在用`Iteraor`遍历`LinkedHashMap`时，先得到的记录肯定是**先插入**的，在遍历的时候会比`HashMap`慢，有`HashMap`的**全部特性**。

#### TreeMap

`TreeMap`实现`SortMap`接口，能够把它保存的记录根据**键排序**，默认是按键值的**升序排序**（自然顺序），也可以指定排序的比较器，当用`Iterator`遍历`TreeMap`时，得到的记录是排过序的。**不允许`key`值为空，非同步**。

* `TreeMap`判断两个元素相等的标准：两个`key`通过`compareTo()`方法返回`0`，则认为这两个`key`相等。
* 如果使用自定义的类来作为`TreeMap`中的`key`值，且想让`TreeMap`能够良好的工作，则必须**重写**自定义类中的`equals()`方法。

### Map遍历

#### KeySet()

将`Map`中所有的**键**存入到`set`集合中。因为`set`具备**迭代器**。所有可以迭代方式取出所有的键，再根据`get`方法。获取每一个键对应的值。`keySet()`:迭代后只能通过`get()`取`key`。取到的结果会**乱序**，是因为取得数据行主键的时候，使用了`HashMap.keySet()`方法，而这个方法返回的`Set`结果，里面的数据是**乱序排放**的。

``` java
    Map map = new HashMap();
    map.put("key1","lisi1");
    map.put("key2","lisi2");
    map.put("key3","lisi3");
    map.put("key4","lisi4");  
    //先获取map集合的所有键的set集合，keyset()
    Iterator it = map.keySet().iterator();
    //获取迭代器
    while(it.hasNext()) {
        Object key = it.next();
        System.out.println(map.get(key));
    }
```

#### entrySet()

``` java
Set<Map.Entry<K,V>> entrySet() //返回此映射中包含的映射关系的 Set 视图。
```

（一个关系就是一个键-值对），就是把`(key-value)`作为一个整体一对一对地存放到`Set`集合当中的。`Map.Entry`表示映射关系。`entrySet()`：迭代后可以`e.getKey()`，`e.getValue()`两种方法来取`key`和`value`。返回的是`Entry`接口。

``` java
    Map map = new HashMap();
    map.put("key1","lisi1");
    map.put("key2","lisi2");
    map.put("key3","lisi3");
    map.put("key4","lisi4");
    //将map集合中的映射关系取出，存入到set集合
    Iterator it = map.entrySet().iterator();
    while(it.hasNext()) {
        Entry e =(Entry) it.next();
        System.out.println("键"+e.getKey () + "的值为" + e.getValue());
    }
```

* `entrySet()`方法，效率较高：
    * `keySet`其实是遍历了`2`次，一次是转为`iterator`，一次就是从`HashMap`中取出`key`所对于的`value`
    * `entryset`只是遍历了`1`次，它把`key`和`value`都放到了`entry`中，所以效率较高

### HashMap一些功能实现

#### 按值排序

`HashMap`按值排序通过`Collections`的`sort`方法，以下为`HashMap`的几种遍历方式

``` java
    //Collection And Map
    public static void testCM(){
        //Collection
        Map<Integer , String> hs = new HashMap<Integer , String>();
        int i = 0;
        hs.put(199, "序号:"+201);
        while(i<50){
            hs.put(i, "序号:"+i);
            i++;
        }
        hs.put(-1, "序号:"+200);
        hs.put(200, "序号:"+200);

        //遍历方式一:for each遍历HashMap的entryset，注意这种方式在定义的时候就必须写成
        //Map<Integer , String> hs，不能写成Map hs;
        for(Entry<Integer , String> entry : hs.entrySet()){
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        }
        //遍历方式二：使用EntrySet的Iterator
        Iterator<Map.Entry<Integer , String>> iterator = hs.entrySet().iterator();
        while(iterator.hasNext()){
            Entry<Integer , String> entry =  iterator.next();
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        };
        //遍历方式三：for each直接使用HashMap的keyset
        for(Integer key : hs.keySet()){
            System.out.println("key:"+key+"  value:"+hs.get(key));
        };
        //遍历方式四：使用keyset的Iterator
        Iterator keyIterator = hs.keySet().iterator();
        while(keyIterator.hasNext()){
            Integer key = (Integer)keyIterator.next();
            System.out.println("key:"+key+"  value:"+hs.get(key));
        }
    }
```

* 使用`keyset`的两种方式都会**遍历两次**，所以效率没有使用`EntrySet`高
* `HashMap`输出是**无序**的，这个无序不是说每次遍历的结果顺序不一样，而是说与**插入顺序不一样**

值排序：

``` java
    //对HashMap排序
    public static void sortHashMap(Map<Integer , String> hashmap){

        System.out.println("排序后");

        //第一步，用HashMap构造一个LinkedList
        Set<Entry<Integer , String>> sets = hashmap.entrySet();
        LinkedList<Entry<Integer , String>> linkedList = new LinkedList<Entry<Integer , String>>(sets);

        //用Collections的sort方法排序
        Collections.sort(linkedList , new Comparator<Entry<Integer , String>>(){

            @Override
            public int compare(Entry<Integer , String> o1, Entry<Integer , String> o2) {
                // TODO Auto-generated method stub
                /*String object1 = (String) o1.getValue();
                String object2 = (String) o2.getValue();
                return object1.compareTo(object2);*/
                return o1.getValue().compareTo(o2.getValue());
            }

        });

        //第三步，将排序后的list赋值给LinkedHashMap
        Map<Integer , String> map = new LinkedHashMap();
        for(Entry<Integer , String> entry : linkedList){
            map.put(entry.getKey(), entry.getValue());
        }
        for(Entry<Integer , String> entry : map.entrySet()){
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        }
    }
```

#### 键排序

1. 用`Collections`的`sort`方法，更改比较规则
2. `TreeMap`是按**键排序**的，默认**升序**，所以可以通过`TreeMap`来实现
    ``` java
    public static void sortHashMapByKey(Map hashmap){

        System.out.println("按键排序后");

        //第一步：先创建一个TreeMap实例，构造函数传入一个Comparator对象。
        TreeMap<Integer , String> treemap = new TreeMap<Integer , String>(new Comparator<Integer>(){

            @Override
            public int compare(Integer o1,Integer o2) {
                // TODO Auto-generated method stub
                return Integer.compare(o1, o2);
            }

        });
        //第二步：将要排序的HashMap添加到我们构造的TreeMap中。
        treemap.putAll(hashmap);
        for(Entry<Integer , String> entry : treemap.entrySet()){
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        }
    }
    ```
3. 可以通过`keyset`取出所有的`key`，然后将`key`排序，再有序的将`key-value`键值对存到`LinkedHashMap`中

#### value去重

对于`HashMap`而言，它的`key`是**不能重复**的，但是它的`value`是**可以重复**的。
将`HashMap`的`key-value`对调，然后赋值给一个新的`HashMap`，由于`key`的不可重复性，此时就将重复值去掉了。最后将新得到的`HashMap`的`key-value`再对调一次即可。

#### HashMap线程同步

1. &nbsp;
    ``` java
    Map<Integer , String> hs = new HashMap<Integer , String>();
        hs = Collections.synchronizedMap(hs);
    ```
2. &nbsp;
    ``` java
    ConcurrentHashMap<Integer , String> hs = new ConcurrentHashMap<Integer , String>();
    ```
### Iterator

`Iterator`，所有的集合类，都**实现**了`Iterator`**接口**，这是一个用于**遍历集合中元素**的接口，主要包含以下三种方法：

1. `boolean hasNext()`是否还有下一个元素。
2. `Object next()`返回下一个元素。
3. `void remove()`删除当前元素。

* 当使用`Iterator`对集合元素进行**迭代**时，`Iterator`并不是把集合元素本身传给了迭代变量，而是把集合元素的**值**传给了迭代变量（就如同参数传递是值传递，基本数据类型传递的是值，引用类型传递的仅仅是对象的引用变量），所以**修改迭代变量的值对集合元素本身没有任何影响**。
* `Iterator`只能**单向移动**
* `Iterator.remove()`是**唯一安全**的方式来在迭代过程中**修改集合**；如果在迭代过程中以任何其它的方式修改了基本集合将会产生未知的行为。而且每调用一次`next()`方法，`remove()`方法只能**被调用一次**，如果违反这个规则将**抛出一个异常**。
* `for-each`底层使用迭代遍历，如果对象可以使用`for-each`，必须实现`Iterable<E>`接口

#### ListIterator

`ListIterator`是一个功能更加强大的迭代器, 它继承于`Iterator`接口,**只能**用于各种`List`类型的访问。可以通过调用`listIterator()`方法产生一个指向`List`**开始处**的`ListIterator`, 还可以调用`listIterator(n)`方法创建一个一开始就指向列表索引为`n`的元素处的`ListIterator`。

``` java
public interface ListIterator<E> extends Iterator<E> {
    boolean hasNext();

    E next();

    boolean hasPrevious();

    E previous();

    int nextIndex();

    int previousIndex();

    void remove();

    void set(E e);

    void add(E e);  
}
```

* **双向移动**（向前/向后遍历）
* 产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引
* 可以使用`set()`方法替换它访问过的**最后一个元素**
* 可以使用`add()`方法在`next()`方法返回的元素之前或`previous()`方法返回的元素之后插入一个元素

### List、Set、Map的区别

* List（有序、可重复）
* Set（无序、不能重复）
* Map（键值对、键唯一、值不唯一）

| &nbsp;     | &nbsp;      | 是否有序      | 是否允许元素重复                              |
| ---------- | ----------- | --------- | ------------------------------------- |
| Collection | &nbsp;      | &nbsp;    | &nbsp;                                |
| List       | &nbsp;      | 是         | 是                                     |
| Set        | AbstractSet | 否         | 否                                     |
| &nbsp;     | HashSet     | 否         | 否                                     |
| &nbsp;     | TreeSet     | 是（用二叉树排序） | 否                                     |
| Map        | AbstractMap | 否         | 使用key-value来映射和存储数据，key必须唯一，value可以重复 |
| &nbsp;     | HashMap     | 否         | 使用key-value来映射和存储数据，key必须唯一，value可以重复 |
| &nbsp;     | TreeMap     | 是（用二叉排序树） | 使用key-value来映射和存储数据，key必须唯一，value可以重复 |

### Vector和ArrayList

1. `vector`是**线程同步**的，所以它也是**线程安全**的，而`arraylist`是**线程异步**的，是**不安全**的。如果不考虑到线程的安全因素，一般用`arraylist`效率比较**高**。
2. 如果集合中的元素的数目大于目前集合数组的长度时，`vector`增长率为目前数组长度的`100%`，而`arraylist`增长率为目前数组长度的`50%`。如果在集合中使用数据量比较大的数据，用`vector`有一定的优势。
3. 如果查找一个**指定位置**的数据，`vector`和`arraylist`使用的时间是相同的，如果**频繁**的访问数据，这个时候使用`vector`和`arraylist`都可以。而如果移动一个指定位置会导致后面的元素都发生移动，这个时候就应该考虑到使用`linklist`,因为它移动一个指定位置的数据时**其它元素不移动**。

`ArrayList`和`Vector`是采用**数组**方式存储数据，此数组元素数**大于**实际存储的数据以便增加和插入元素，都允许直接序号索引元素，但是插入数据要涉及到数组元素移动等内存操作，所以**索引数据快**，**插入数据慢**，`Vector`由于使用了`synchronized`方法（线程安全）所以性能上比`ArrayList`要差，`LinkedList`使用**双向链表**实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项的前后项即可，所以插入数度较快。

### arraylist和linkedlist

1. `ArrayList`是实现了基于**动态数组**的数据结构，`LinkedList`基于**链表**的数据结构。
2. 对于随机访问`get`和`set`，`ArrayList`优于`LinkedList`，因为`LinkedList`要移动指针。
3. 对于新增和删除操作`add`和`remove`，`LinkedList`比较占优势，因为`ArrayList`要移动数据。这一点要看实际情况的。若只对**单条数据**插入或删除，`ArrayList`的速度反而优于`LinkedList`。但若是**批量随机**的插入删除数据，`LinkedList`的速度大大优于`ArrayList`，因为`ArrayList`每插入一条数据，要移动插入点及之后的所有数据。

### HashMap与TreeMap

1. `HashMap`通过`hashcode`对其内容进行快速查找，而`TreeMap`中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用`TreeMap`（`HashMap`中元素的排列顺序是不固定的）。
2. 在`Map`中插入、删除和定位元素，`HashMap`是最好的选择。但如果要按自然顺序或自定义顺序遍历键，那么`TreeMap`会更好。使用`HashMap`要求添加的键类明确定义了`hashCode()`和`equals()`的实现。

两个`map`中的元素一样，但顺序不一样，导致`hashCode()`不一样。
同样做测试：
在`HashMap`中，同样的值的`map`,顺序不同，`equals`时，`false`;
而在`treeMap`中，同样的值的`map`,顺序不同,`equals`时，`true`，说明，`treeMap`在`equals()`时是整理了顺序的。

### HashTable与HashMap

1. 同步性:`Hashtable`是**线程安全**的，也就是说是同步的，而`HashMap`是**线程序不安全**的，不是同步的。
2. `HashMap`允许存在**一个**为`null`的`key`，**多个**为`null`的`value` 。
3. `hashtable`的`key`和`value`都不允许为`null`。

### Collection和Collections

1. `java.util.Collection`是一个**集合接口**（集合类的一个**顶级接口**）。它提供了对集合对象进行基本操作的通用接口方法。`Collection`接口的意义是为各种具体的集合提供了最大化的统一操作方式，其直接继承接口有`List`与`Set`。
    ``` mermaid
    graph TD
     Collection-->List
     Collection-->Set
     List-->LinkedList
     List-->ArrayList
     List-->Vector
     Vector-->Stack
    ```
2. `java.util.Collections`是一个**包装类**（工具类/帮助类）。它包含有各种有关**集合操作**的**静态多态**方法。**此类不能实例化**，就像一个工具类，用于对集合中元素进行**排序**、**搜索**以及**线程安全**等各种操作，服务于Java的`Collection`框架。
    ``` java
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;
  
    public class TestCollections {
        public static void main(String args[]) {
            //注意List是实现Collection接口的
            List list = new ArrayList();
            double array[] = { 112, 111, 23, 456, 231 };
            for (int i = 0; i < array.length; i++) {
                list.add(new Double(array[i]));
            }
            Collections.sort(list);
            for (int i = 0; i < array.length; i++) {
                System.out.println(list.get(i));
            }
            // 结果：23.0 111.0 112.0 231.0 456.0
        }
    }
    ```

## 泛型

### 擦除

在编译期间，所有的泛型信息都会被擦除，`List<Integer>`和`List<String>`类型，在编译后都会变成`List`类型（原始类型）。

### 泛型上限

> 接受数字类型的集合参数并遍历元素

``` java
public static void m(List<? extends Number> nubmers) {
    ...
}
```

* `?`通配符
* `?` `extends` `Number` 可以接受`Number`类及其子类
* `?` `extends` 类/接口或者/子类/子接口对象
* 上限不可以对集合进行增删操作

### 泛型下限

> 接受`String`或其父类的集合参数

``` java
public static void n(List<? super String> numbers) {
    ...
}
```

* `?` `extends` `String` 可以接受`String`类或父类
* 上下限不能同时出现
* 下限可以对集合进行增删操作(`String`)

## 文件

### 路径

* 绝对路径：以盘符或者是`/`开头的路径
    * D:\\aa.txt
    * /home/software 
    * 和当前所处的路径没有任何关系，直接定位到指定的路径
* 相对路径：不以盘符或者是`/`开头的路径
    * `a.txt`
    * 需要以当前的位置来计算另一个位置，路径发生改变的时候能够自动计算

## IO

* 输入流指数据从外部流向程序
* 输出流指数据从程序流向外部

根据数据的传输方向：输入流、输出流
根据数据的传输形式：字符流、字节流
 
&nbsp;|字节流|字符流
-|-|-   
输入流|InputStream|Reader	   
输出流|OutputStream|Writer

* 四个基本流都是抽象类，不能直接创建对象

### 流中的异常处理

1. 将流对象放到`try`之外声明并且赋值为`null`，放到`try`之内创建
2. 关流之前需要判断流对象是否初始化成功 --- 判断流对象是否为`null`
3. 关流之后需要将流对象设置为`null`
4. 为了防止关流失败导致数据丢失，需要在写外之后手动冲刷一次缓冲区

### 缓冲流

`BufferedReader`：提供了缓冲区，能够实现按行读取的效果。
利用`FileReader`来构建了`BufferedReader`，然后再`BufferedReader`对读取功能做了增强，这种方式称之为**装饰设计模式**：利用了**同类对象**构建自己对象本身，对对象身上的功能做了增强或者改善

`BufferedWriter`提供了一个更大的缓冲区
* 注意：`Java`中的原生的字符流只能操作字符类文件 `txt` `java` `html`等，但是不能读取`office`组件

### 字节流

`FileInputStream`：字节输入流，可以从文件中读取数据
`FileOutputStream`：字节输出流，可以向文件中写数据

* 一般缓冲区的合适大小是10-15M

### 系统流/标准流

系统流都是字节流
`System.in`：标准输入流
`System.out`：标准输出流
`System.err`：标准错误流

### 转换流

字符流和字节流之间用的转换就是转换流。
`OutputStreamWriter`：将字符流转化为字节流 `FileWriter`是它的子类
`InputStreamReader`：将字节流转化为字符流 `FileReader`是它的子类 

### 合并流

`SequenceInputStream`：创建输入流分别指向对应的文件，然后需要创建一个`Vector`集合存放这些输入流，利用`Vector`集合来产生一个`Enumeration`对象，使用`Enumeration`对象来构建合并流，最后利用合并流来读取数据进行合并。

### 序列化/反序列化流

序列化：将对象转化为字节之后进行存储
持久化：将数据存储在硬盘上
反序列化：将字节转化为对象的过程

* 一个对象想要被序列化，那么它所对应的类必须实现接口`Serializable`，这个接口中没有任何的方法和属性，仅仅起**标志性作用**
* 用`static/transient`修饰的属性不会被序列化

如果一个类产生的对象允许被序列化，那么这个时候这个类在编译的时候会根据当前类中的属性自动计算一个**版本号**。当反序列化的时候，拿着对象中的版本号和类中版本号做比较，如果相等，则说明这个对象是这个产生的，可以被反序列化。如果没有手动指定版本号，自动计算版本号，那么就意味着**类每变动一次**，**版本号**就要**重新计算一次**。为了让序列化出去的对象反序列化回来，需要**手动指定版本号** --- 

``` java
private static final long serialVersionUID
```

### Properties

是一个可以**被持久化**的**映射**。键和值的类型都是`String`
`properties`文件的默认编码就是**西欧编码**，当向`properties`文件中存放中文的时候变成了对应的`Unicode`编码

### 单元测试

``` java
import org.junit.Test;
```

* 没有参数
* 无返回值
* 非静态方法

## 线程

> 进程的执行在**宏观上并行**，在**微观上串行**。

### 线程的定义

1. 写一个类继承`Thread`类，将要执行的逻辑放到`run`方法中，创建线程对象，调用`start`方法来启动线程执行任务。
2. 写一个类实现`Runnable`接口，重写`run`方法，创建`Runnable`对象，然后将`Runnable`对象作为参数传递到`Thread`对象中，利用`Thread`对象来启动线程。
3. 写一个类实现`Callable`接口，重写`call`方法。

### 多线程的并发安全问题

在多个线程同时执行时，相互抢占资源导致出现了不合常理的数据现象——多线程的并发安全问题。

> 多线程在执行的时候是相互抢占，而且抢占是发生在线程执行的每一步过程中。

### 同步锁机制

> 利用`synchronized`同步代码块解决多线程并发安全问题。

* **同步**：一段逻辑在同一时间**只能有一个**线程执行
* **异步**：一段逻辑在同一时间能**有多个**线程执行
* 同步一定安全，安全不一定同步
* 异步不一定不安全，不安全一定异步

> 从微观上而言，同步一定是安全的，安全也一定是同步的。从宏观上，同步一定是安全的，安全不一定是同步的。

* **锁对象**：要求锁对象要被所有的线程都认识
    * **共享对象**
    * **类的字节码**
        * 方法区被线程所共享
    * `this`
        * 同一个对象开启多个线程

> 堆内存和方法区被线程共享

> 栈内存，本地方法栈，PC计数器是每个线程所独有的

* 如果同步方法是一个**非静态**方法，那么以`this`作为锁对象
* 如果同步方法是一个**静态**方法，那么以当前类作为锁对象

### 死锁

由于锁之间相互嵌套并且锁对象不同导致线程之间相互锁死，致使代码无法继续往下执行

* 避免死锁：统一锁对象，减少锁的嵌套

### 活锁

这个资源没有被任何的线程持有占用，导致程序无法往下执行

### 等待唤醒机制

使用`wait`和`notify`来调节线程的执行顺序

> 等待唤醒机制必须结合锁来使用，而且锁对象是谁就用谁进行等待唤醒

### 线程的优先级

线程的优先等级为：1-10
理论上，数字越大优先级越高，抢占到资源的概率就越大
实际上，相邻的两个优先级的差别非常不明显。如果优先级差到**5个单位及以上**，则结果会相对明显一点点

### 守护线程

守护别的线程。**只要被守护的线程结束，那么无论守护线程完成与否都会结束。**
在线程中，一个线程要么是**守护线程**，要么是**被守护的线程**。

> 当最后一个被守护的线程结束才会导致所有的守护线程结束——`GC`

### 启动线程

1. 用户请求创建
2. 系统自启
3. 被其他线程启动

### 线程结束

1. 正常执行结束
2. 被其他请求强制结束
3. 执行中出现异常或者错误

### 单例模式

* 设计模式：在软件开发工程中使用的常见的解决问题的方式

> 在全局中只存在一个实例的现象

* 饿汉式：会增加类的加载时间，能够避免并发问题
* 懒汉式：减少加载时间，会导致多线程的并发安全问题


