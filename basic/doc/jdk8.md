# jdk1.8新特性

jdk1.8新增特性如下：

* lambda表达式
* 函数式接口
* 方法引用和构造器调用
* Stream API
* 接口中的默认方法和静态方法
* 新时间日期API

**hashMap数据结构的优化**

原hashMap采用的数据结构为哈希表（数组+链表）。

默认大小为16，一个下标0-15的数组。

存储元素时，需要调用元素的`hashcode`方法计算其哈希值，使用哈希值作为数组索引。如果对应索引处为空，直接存放元素，如果已有元素，使用`equals`方法比较元素内容。如果内容相同，**值覆盖**，如果内容不同：

> 1.7

新的元素放在前面，形成链表，发生**碰撞**。

加载因子：0.75。每当数组现容量达到总容量的75%时进行扩容。

> 1.8

使用数组 + 链表 + 红黑树实现hashMap。当

$$碰撞元素个数 > 8 \ \& \ 总容量 > 64$$

引入红黑树。

链表新元素加到末尾。

## lambda表达式

> 本质是一段匿名内部类，是一段可以传递的代码。

**例如**

``` java
// 匿名内部类
Comparator<Integer> cpt = new Comparator<Integet>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
};

TreeSet<Integer> set = new TreeSet<>(cpt);

System.out.println("-------");

// 使用lambda表达式
Comparator<Integer> cpt2 = (x, y) -> Integer.compate(x, y);
TreeSet<Integer> set2 = new TreeSet<>(cpt2);
```

**语法总结**

| 前置                                       | 语法                                                 |
|--------------------------------------------|------------------------------------------------------|
| 无参数无返回值                             | () -> System.out.println(“Hello WOrld”)              |
| 有一个参数无返回值                         | (x) -> System.out.println(x)                         |
| 有且只有一个参数无返回值                   | x -> System.out.println(x)                           |
| 有多个参数，有返回值，有多条lambda体语句   | (x，y) -> {System.out.println(“xxx”);return xxxx;}； |
| 有多个参数，有返回值，只有一条lambda体语句 | (x，y) -> xxxx                                       |

* 当一个接口中存在多个抽象方法时，如果使用lambda表达式，并不能智能匹配对应的抽象方法，因此引入函数式接口的概念。

## 函数式接口

> 只定义一个抽象方法的接口，并且提供注解`@FunctionalInterface`

**常见四大函数式接口**

* `Consumer<T>`：消费型接口，有参无返回值

    ``` java
    @Test
    public void test() {
        changeStr("test", (str) -> System.out.println(str));
    }

    /**
     * Consumer<T> 消费型接口
     * @param str
     * @param con
     */
    public void changeStr(String str, Consumer<String> con) {
        con.accept(str);
    }
    ```

* `Supplier<T>`：供给型接口，无参有返回值

    ```java
    @Test
    public void Test2() {
        String value = getValue(() -> "test");
        System.out.println(value);
    }

    /**
     * Supplier<T> 供给型接口
     * @param sup
     * @return
     */
    public String getValue(Supplier<String> sup) {
        return sup.get();
    }  
    ```

* `Function<T, R>`：函数式接口，有参有返回值

    ``` java
    @Test
    public void test3() {
        Long result = changeNum(100L, (x) -> x + 200L);
        System.out.println(result);
    }

    /**
     * Function<T, R> 函数式接口
     * @param num
     * @param fun
     * @return
     */
    public Long changeNum(Long num, Function<Long, Long> fun) {
        return fun.apply(num);
    }
    ```

* `Predicate<T>`：断言型接口，有参有返回值，返回值是boolean类型

    ``` java
    @Test
    public void test4() {
        boolean result = changeBoolean("test", (str) -> str.length() > 5);
        System.out.println(result);
    }

    /**
     * Predicate<T> 断言型接口
     * @param str
     * @param pre
     * @return
    */
    public boolean changeBoolean(String str, predicate<String> pre) {
        return pre.test(str);
    }
    ```

## 方法引用

> lambda体中的内容有方法已经实现，可以使用方法引用。

### 方法引用

**三种表现形式**

1. 对象::实例方法名
2. 类::静态方法名
3. 类::实例方法名（lambda参数列表中第一个参数是实例方法的调用者，第二个参数是实例方法的参数时可用）

``` java
public void test() {
    /**
     *注意：
     *   1.lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值类型保持一致！
     *   2.若lambda参数列表中的第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时，可以使ClassName::method
     */
    Consumer<Integer> con = (x) -> System.out.println(x);
    con.accept(100);

    // 方法引用-对象::实例方法
    Consumer<Integer> con2 = System.out::println;
    con2.accept(200);

    // 方法引用-类名::静态方法名
    BiFunction<Integer, Integer, Integer> biFun = (x, y) -> Integer.compare(x, y);
    BiFunction<Integer, Integer, Integer> biFun2 = Integer::compare;
    Integer result = biFun2.apply(100, 200);

    // 方法引用-类名::实例方法名
    BiFunction<String, String, Boolean> fun1 = (str1, str2) -> str1.equals(str2);
    BiFunction<String, String, Boolean> fun2 = String::equals;
    Boolean result2 = fun2.apply("hello", "world");
    System.out.println(result2);
}
```

### 构造器引用

`ClassName::new`

``` java
public void test2() {
    // 构造方法引用  类名::new
    Supplier<Employee> sup = () -> new Employee();
    System.out.println(sup.get());
    Supplier<Employee> sup2 = Employee::new;
    System.out.println(sup2.get());

    // 构造方法引用 类名::new （带一个参数）
    Function<Integer, Employee> fun = (x) -> new Employee(x);
    Function<Integer, Employee> fun2 = Employee::new;
    System.out.println(fun2.apply(100));
}
```

### 数组引用

`Type[]::new`

``` java
public void test(){
    // 数组引用
    Function<Integer, String[]> fun = (x) -> new String[x];
    Function<Integer, String[]> fun2 = String[]::new;
    String[] strArray = fun2.apply(10);
    Arrays.stream(strArray).forEach(System.out::println);
}
```

## Stream API

**步骤**

* 创建Stream
* 中间操作（过滤、map）
* 终止操作
