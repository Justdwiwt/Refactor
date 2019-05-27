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
| ------------------------------------------ | ---------------------------------------------------- |
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

**Stream的创建**

``` java
// 1，校验通过Collection 系列集合提供的stream()或者paralleStream()
List<String> list = new ArrayList<>();
Strean<String> stream1 = list.stream();

// 2.通过Arrays的静态方法stream()获取数组流
String[] str = new String[10];
Stream<String> stream2 = Arrays.stream(str);

// 3.通过Stream类中的静态方法of
Stream<String> stream3 = Stream.of("aa","bb","cc");

// 4.创建无限流
// 迭代
Stream<Integer> stream4 = Stream.iterate(0,(x) -> x+2);

//生成
Stream.generate(() -> Math.random());
```

**Stream的中间操作**

``` java
/**
 * 筛选 过滤  去重
 */
emps.stream()
    .filter(e -> e.getAge() > 10)
    .limit(4)
    .skip(4)
    // 需要流中的元素重写hashCode和equals方法
    .distinct()
    .forEach(System.out::println);


/**
 *  生成新的流 通过map映射
 */
emps.stream()
    .map((e) -> e.getAge())
    .forEach(System.out::println);


/**
 *  自然排序  定制排序
 */
emps.stream()
    .sorted((e1 ,e2) -> {
        if (e1.getAge().equals(e2.getAge())){
            return e1.getName().compareTo(e2.getName());
        } else{
            return e1.getAge().compareTo(e2.getAge());
        }
    })
    .forEach(System.out::println);
```

**Stream的终止操作**

``` java
/**
 *      查找和匹配
 *          allMatch-检查是否匹配所有元素
 *          anyMatch-检查是否至少匹配一个元素
 *          noneMatch-检查是否没有匹配所有元素
 *          findFirst-返回第一个元素
 *          findAny-返回当前流中的任意元素
 *          count-返回流中元素的总个数
 *          max-返回流中最大值
 *          min-返回流中最小值
 */

/**
 *  检查是否匹配元素
 */
boolean b1 = emps.stream().allMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
System.out.println(b1);

boolean b2 = emps.stream().anyMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
System.out.println(b2);

boolean b3 = emps.stream().noneMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
System.out.println(b3);

Optional<Employee> opt = emps.stream().findFirst();
System.out.println(opt.get());

// 并行流
Optional<Employee> opt2 = emps.parallelStream().findAny();
System.out.println(opt2.get());

long count = emps.stream().count();
System.out.println(count);

Optional<Employee> max = emps.stream().max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
System.out.println(max.get());

Optional<Employee> min = emps.stream().min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
System.out.println(min.get());
```

**`reduce`**

> 可以将流中元素反复结合起来，得到一个值

``` java
/**
 *  reduce ：规约操作
 */
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
Integer count2 = list.stream().reduce(0, (x, y) -> x + y);
System.out.println(count2);

Optional<Double> sum = emps.stream().map(Employee::getSalary).reduce(Double::sum);
System.out.println(sum);
```

**`collect`**

> 将流转换为其他形式，接受一个`Collection`接口的实现，用于给Stream中元素做汇总的方法

``` java
/**
 *  collect：收集操作
 */

List<Integer> ageList = emps.stream().map(Employee::getAge).collect(Collectors.toList());
ageList.stream().forEach(System.out::println);
```

**并行流和串行流**

并行流将内容切割成多个数据块，并且使用多个线程分别处理每个数据块的内容。

Stream API 中声明可以通过`parallel()`与`sequential()`方法在并行流和串行流之间进行切换。

**Fork/Join框架**

> 在必要情况下，将一个大任务拆分（fork）成若干个小任务（不可再拆分），再将这些小任务运算的结果进行汇总（join）。

_即递归分合、分而治之。_

**工作窃取模式（work-stealing）：**

当执行新的任务时，它可以将其拆分分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。

相对于一般的线程池实现，fork/join框架的优势体现在对其中包含的任务的处理方式上。

在一般的线程池中，如果一个线程正在执行的任务由于某些原因无法继续运行，那么该线程会处于等待状态。而在fork/join中，如果某个子问题由于等待另一个子问题的完成而无法继续运行，那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行。这种方式减少了线程的等待时间，提高了性能。

``` java
/**
 * 要想使用Fark—Join，类必须继承
 * RecursiveAction（无返回值）
 * Or
 * RecursiveTask（有返回值）
 */
public class ForkJoin extends RecursiveTask<Long> {

    /**
     * 要想使用Fark—Join，类必须继承RecursiveAction（无返回值） 或者
     * RecursiveTask（有返回值）
     */
    private static final long serialVersionUID = 23423422L;

    private long start;
    private long end;

    public ForkJoin() {
    }

    public ForkJoin(long start, long end) {
        this.start = start;
        this.end = end;
    }

    // 定义阙值
    private static final long THRESHOLD = 10000L;

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (long i = start; i < end; i++) {
                sum += i;
            }
            return sum;
        } else {
            long middle = (end - start) / 2;
            ForkJoin left = new ForkJoin(start, middle);
            //拆分子任务，压入线程队列
            left.fork();
            ForkJoin right = new ForkJoin(middle + 1, end);
            right.fork();

            //合并并返回
            return left.join() + right.join();
        }
    }

    /**
     * 实现数的累加
     */
    @Test
    public void test1() {
        //开始时间
        Instant start = Instant.now();

        //这里需要一个线程池的支持
        ForkJoinPool pool = new ForkJoinPool();

        ForkJoinTask<Long> task = new ForkJoin(0L, 10000000000L);
        // 没有返回值     pool.execute();
        // 有返回值
        long sum = pool.invoke(task);

        //结束时间
        Instant end = Instant.now();
        System.out.println(Duration.between(start, end).getSeconds());
    }

    /**
     * java8 并行流 parallel()
     */
    @Test
    public void test2() {
        //开始时间
        Instant start = Instant.now();

        // 并行流计算    累加求和
        LongStream.rangeClosed(0, 10000000000L).parallel().reduce(0, Long :: sum);

        //结束时间
        Instant end = Instant.now();
        System.out.println(Duration.between(start, end).getSeconds());
    }

    @Test
    public void test3(){
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
        list.stream().forEach(System.out::print);

        list.parallelStream().forEach(System.out::print);
    }
```

**展示多线程的效果：**

``` java
@Test
public void test(){
    // 并行流 多个线程执行
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
    numbers.parallelStream().forEach(System.out::print);

    //
    System.out.println("=========================");
    numbers.stream().sequential().forEach(System.out::print);
}
```

## Optional容器

> 使用Optional容器可以快速的定位NPE，并且在一定程度上可以减少对参数非空检验的代码量。

``` java
/**
 *      Optional.of(T t); // 创建一个Optional实例
 *      Optional.empty(); // 创建一个空的Optional实例
 *      Optional.ofNullable(T t); // 若T不为null，创建一个Optional实例，否则创建一个空实例
 *      isPresent();    // 判断是够包含值
 *      orElse(T t);   //如果调用对象包含值，返回该值，否则返回T
 *      orElseGet(Supplier s);  // 如果调用对象包含值，返回该值，否则返回s中获取的值
 *      map(Function f): // 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty();
 *      flatMap(Function mapper);// 与map类似。返回值是Optional
 *
 *      总结：Optional.of(null)  会直接报NPE
 */

Optional<Employee> op = Optional.of(new Employee("zhansan", 11, 12.32, Employee.Status.BUSY));
System.out.println(op.get());

// NPE
Optional<Employee> op2 = Optional.of(null);
System.out.println(op2);

@Test
public void test2(){
    Optional<Object> op = Optional.empty();
    System.out.println(op);

    // No value present
    System.out.println(op.get());
}

@Test
public void test3(){
    Optional<Employee> op = Optional.ofNullable(new Employee("lisi", 33, 131.42, Employee.Status.FREE));
    System.out.println(op.get());

    Optional<Object> op2 = Optional.ofNullable(null);
    System.out.println(op2);
    // System.out.println(op2.get());
}

@Test
public void test5(){
    Optional<Employee> op1 = Optional.ofNullable(new Employee("张三", 11, 11.33, Employee.Status.VOCATION));
    System.out.println(op1.orElse(new Employee()));
    System.out.println(op1.orElse(null));
}

@Test
public void test6(){
    Optional<Employee> op1 = Optional.of(new Employee("田七", 11, 12.31, Employee.Status.BUSY));
    op1 = Optional.empty();
    Employee employee = op1.orElseGet(() -> new Employee());
    System.out.println(employee);
}

@Test
public void test7(){
    Optional<Employee> op1 = Optional.of(new Employee("田七", 11, 12.31, Employee.Status.BUSY));
    System.out.println(op1.map( (e) -> e.getSalary()).get());
}
```

## 接口中可以定义默认实现方法和静态方法

> 在接口中可以使用default和static关键字来修饰接口中定义的普通方法。

``` java
public interface Interface {
    default  String getName(){
        return "zhangsan";
    }

    static String getName2(){
        return "zhangsan";
    }
}
```

_在JDK1.8中很多接口会新增方法，为了保证1.8向下兼容，1.7版本中的接口实现类不用每个都重新实现新添加的接口方法，引入了default默认实现，static的用法是直接用接口名去调方法即可。当一个类继承父类又实现接口时，若后两者方法名相同，则优先继承父类中的同名方法，即“类优先”，如果实现两个同名方法的接口，则要求实现类必须手动声明默认实现哪个接口中的方法。_

## 日期API LocalDate|LocalTime|LocalDateTime

> 新的日期API都是不可变的，更多用于多线程的使用环境中。

``` java
@Test
public void test(){
    // 从默认时区的系统时钟获取当前的日期时间。不用考虑时区差
    LocalDateTime date = LocalDateTime.now();
    //2018-07-15T14:22:39.759
    System.out.println(date);

    System.out.println(date.getYear());
    System.out.println(date.getMonthValue());
    System.out.println(date.getDayOfMonth());
    System.out.println(date.getHour());
    System.out.println(date.getMinute());
    System.out.println(date.getSecond());
    System.out.println(date.getNano());

    // 手动创建一个LocalDateTime实例
    LocalDateTime date2 = LocalDateTime.of(2017, 12, 17, 9, 31, 31, 31);
    System.out.println(date2);
    // 进行加操作，得到新的日期实例
    LocalDateTime date3 = date2.plusDays(12);
    System.out.println(date3);
    // 进行减操作，得到新的日期实例
    LocalDateTime date4 = date3.minusYears(2);
    System.out.println(date4);
}
```

``` java
@Test
public void test2(){
    // 时间戳  1970年1月1日00：00：00 到某一个时间点的毫秒值
    // 默认获取UTC时区
    Instant ins = Instant.now();
    System.out.println(ins);

    System.out.println(LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli());
    System.out.println(System.currentTimeMillis());

    System.out.println(Instant.now().toEpochMilli());
    System.out.println(Instant.now().atOffset(ZoneOffset.ofHours(8)).toInstant().toEpochMilli());
}
```

``` java
@Test
public void test3(){
    // Duration:计算两个时间之间的间隔
    // Period：计算两个日期之间的间隔

    Instant ins1 = Instant.now();

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    Instant ins2 = Instant.now();
    Duration dura = Duration.between(ins1, ins2);
    System.out.println(dura);
    System.out.println(dura.toMillis());

    System.out.println("======================");
    LocalTime localTime = LocalTime.now();
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    LocalTime localTime2 = LocalTime.now();
    Duration du2 = Duration.between(localTime, localTime2);
    System.out.println(du2);
    System.out.println(du2.toMillis());
}
```

``` java
@Test
public void test4(){
    LocalDate localDate =LocalDate.now();

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    LocalDate localDate2 = LocalDate.of(2016,12,12);
    Period pe = Period.between(localDate, localDate2);
    System.out.println(pe);
}
```

``` java
@Test
public void test5(){
    // temperalAdjust 时间校验器
    // 例如获取下周日  下一个工作日
    LocalDateTime ldt1 = LocalDateTime.now();
    System.out.println(ldt1);

    // 获取一年中的第一天
    LocalDateTime ldt2 = ldt1.withDayOfYear(1);
    System.out.println(ldt2);
    // 获取一个月中的第一天
    LocalDateTime ldt3 = ldt1.withDayOfMonth(1);
    System.out.println(ldt3);

    LocalDateTime ldt4 = ldt1.with(TemporalAdjusters.next(DayOfWeek.FRIDAY));
    System.out.println(ldt4);

    // 获取下一个工作日
    LocalDateTime ldt5 = ldt1.with((t) -> {
        LocalDateTime ldt6 = (LocalDateTime)t;
        DayOfWeek dayOfWeek = ldt6.getDayOfWeek();
        if (DayOfWeek.FRIDAY.equals(dayOfWeek)){
            return ldt6.plusDays(3);
        }
        else if (DayOfWeek.SATURDAY.equals(dayOfWeek)){
            return ldt6.plusDays(2);
        }
        else {
            return ldt6.plusDays(1);
        }
    });
    System.out.println(ldt5);
}
```

``` java
@Test
public void test6(){
    // DateTimeFormatter: 格式化时间/日期
    // 自定义格式
    LocalDateTime ldt = LocalDateTime.now();
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
    String strDate1 = ldt.format(formatter);
    String strDate = formatter.format(ldt);
    System.out.println(strDate);
    System.out.println(strDate1);

    // 使用api提供的格式
    DateTimeFormatter dtf = DateTimeFormatter.ISO_DATE;
    LocalDateTime ldt2 = LocalDateTime.now();
    String strDate3 = dtf.format(ldt2);
    System.out.println(strDate3);

    // 解析字符串to时间
    DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    LocalDateTime time = LocalDateTime.now();
    String localTime = df.format(time);
    LocalDateTime ldt4 = LocalDateTime.parse("2017-09-28 17:07:05",df);
    System.out.println("LocalDateTime转成String类型的时间："+localTime);
    System.out.println("String类型的时间转成LocalDateTime："+ldt4);
}
```

``` java
// ZoneTime  ZoneDate       ZoneDateTime
@Test
public void test7(){
    LocalDateTime now = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
    System.out.println(now);

    LocalDateTime now2 = LocalDateTime.now();
    ZonedDateTime zdt = now2.atZone(ZoneId.of("Asia/Shanghai"));
    System.out.println(zdt);

    Set<String> set = ZoneId.getAvailableZoneIds();
    set.stream().forEach(System.out::println);
}
```

**新日期API的优点：**

1. `java.util.Date`月份从0开始，需要手动+1，`java.time.LocalDate`月份和星期改成了enum。
2. `java.util.Date`和`SimpleDateFormat`都不是线程安全的，而`LocalDate`和`LocalTime`和`String`一样，是不变类型，线程安全，且不能修改。
3. `java.util.Date`是一个万能接口，包含日期、时间、还有毫秒数，更加明确需求取舍。
4. `java.util.Date`配合`Calendar`操纵日期时间较为繁琐。

> LocalDate

``` java
public static void localDateTest() {

    //获取当前日期,只含年月日 固定格式 yyyy-MM-dd    2018-05-04
    LocalDate today = LocalDate.now();

    // 根据年月日取日期，5月就是5，
    LocalDate oldDate = LocalDate.of(2018, 5, 1);

    // 根据字符串取：默认格式yyyy-MM-dd，02不能写成2
    LocalDate yesteday = LocalDate.parse("2018-05-03");

    // 如果不是闰年 传入29号也会报错
    LocalDate.parse("2018-02-29");
}
```

> LocalDate常用转化

``` java
/**
 * 日期转换常用,第一天或者最后一天...
 */
public static void localDateTransferTest(){
    //2018-05-04
    LocalDate today = LocalDate.now();
    // 取本月第1天： 2018-05-01
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
    // 取本月第2天：2018-05-02
    LocalDate secondDayOfThisMonth = today.withDayOfMonth(2);
    // 取本月最后一天，再也不用计算是28，29，30还是31： 2018-05-31
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
    // 取下一天：2018-06-01
    LocalDate firstDayOf2015 = lastDayOfThisMonth.plusDays(1);
    // 取2018年10月第一个周三 so easy?：  2018-10-03
    LocalDate thirdMondayOf2018 = LocalDate.parse("2018-10-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.WEDNESDAY));
}
```

> LocalTime

``` java
public static void localTimeTest(){
    //16:25:46.448(纳秒值)
    LocalTime todayTimeWithMillisTime = LocalTime.now();
    //16:28:48 不带纳秒值
     LocalTime todayTimeWithNoMillisTime = LocalTime.now().withNano(0);
    LocalTime time1 = LocalTime.parse("23:59:59");
}
```

> LocalDateTime

``` java
public static void localDateTimeTest(){
    //转化为时间戳  毫秒值
    long time1 = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
    long time2 = System.currentTimeMillis();

    //时间戳转化为localdatetime
    DateTimeFormatter df= DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss.SSS");

    System.out.println(df.format(LocalDateTime.ofInstant(Instant.ofEpochMilli(time1),ZoneId.of("Asia/Shanghai"))));
}
```
