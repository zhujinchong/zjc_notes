# 函数式接口

## 行为参数化

函数式编程可以简化代码，提交代码编程效率，是一种思想；Lambda表达式用来简化代码，是一种语法。

下面讲解函数式编程、Lambda表示式、行为参数化概念。

现有两个需求：

* 筛选出重要大于150的苹果；
* 筛选出颜色为green的苹果

解决方法：

* 以下代码使用了策略设计模式，筛选策略是：重量、颜色；
* 将策略封装起来，用的时候选择其中一个。
* 我们将筛选策略以参数的形式传递给filterApples方法，多种行为/策略，一个参数，这就叫行为参数化。

> 行为参数化实现方法有：参数化方法、参数化类、匿名类、Lambda表达式。

```
public interface ApplePredicate{ 
 boolean test (Apple apple); 
}
public class AppleHeavyWeightPredicate implements ApplePredicate{ 
    public boolean test(Apple apple){ 
     return apple.getWeight() > 150; 
    } 
}
public class AppleGreenColorPredicate implements ApplePredicate{
    public boolean test(Apple apple){ 
     return "green".equals(apple.getColor()); 
    } 
}
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){ 
    List<Apple> result = new ArrayList<>(); 
    for(Apple apple: inventory){ 
        if(p.test(apple)){ 
         result.add(apple); 
        }
    } 
    return result; 
} 
// 调用
// inventory是一个List，存的是苹果的数据
List<Apple> redApples = filterApples(inventory, new AppleHeavyWeightPredicate());
List<Apple> redApples = filterApples(inventory, new AppleGreenColorPredicate());
```

优化代码1：这些策略可以以匿名类的形式创建

```
// 调用
List<Apple> redApples = filterApples(inventory, new ApplePredicate() { 
    public boolean test(Apple a){ 
     return "red".equals(a.getColor()); 
    } 
}); 
```

优化代码2：还可以用Lambda表达式

```
// 调用
List<Apple> redApples = filterApples(inventory, (Apple a)->"red".equals(a.getColor())); 
```

优化代码3：将参数用泛型表示

```
public interface Predicate<T>{
 boolean test(T t);
} 
public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> result = new ArrayList<>(); 
    for(T e: list){ 
        if(p.test(e)){ 
         result.add(e);
        }
    }
    return result; 
}
List<Apple> redApples = filter(inventory, (Apple a) -> "red".equals(a.getColor()));
```

行为参数化例子1：排序

```
public interface Comparator<T> {
 public int compare(T o1, T o2);
}
// 匿名类写法
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
     return a1.getWeight().compareTo(a2.getWeight());
    }
});
// Lambda表示式写法
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

行为参数化例子2：线程

```
public interface Runnable{
 public void run();
}
Thread t = new Thread(new Runnable() {
    public void run(){
     System.out.println("Hello world");
    }
});
Thread t = new Thread(() -> System.out.println("Hello world"));
```

## 函数式接口

函数式接口：只有一个抽象方法的接口。

上面的Predicate/Comparator/Runnable都是函数式接口，如何设计自己需要的函数式接口？

```
1. 定义函数描述：如(int, int)->int
2. 定义函数式接口：
    public interface IntBinaryOperator(){
        int applyAsInt(int a, int b);
    }
3. 执行行为
 public static int binaryOp(int a, int b, IntBinaryOperator op){
  return op.applyAsInt(a, b);
 }
4. 传递Lambda
 binaryOp(a, b, (a, b)->{return a-b;})
 binaryOp(a, b, (a, b)->{return a+b;})
```

## 方法引用（语法糖）

方法引用案例：

```
之前：lambda表达式
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
现在：
inventory.sort(comparing(Apple::getWeight));
```

方法引用是Lambda表示式的快捷写法，也就是语法糖，通常Lambda表示更容易理解。方法引用一般有三种：

```
Lambda: (args) -> ClassName.staticMethod(args)
方法引用: ClassName::staticMethod

Lambda: (objectName, args) -> objectName.instanceMethod(args)
方法引用: ClassName::instanceMethod

Lambda: (args) -> objectName.instanceMethod(args)
方法引用: objectName::instanceMethod
```

方法引用扩展：

```
Comparator具有一个叫作comparing的静态方法，它可以接受一个Function来提取Comparable键值，并生成一个Comparator对象：
 Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
所以，还可以这样用：
 inventory.sort(Comparator.comparing((a) -> a.getWeight())); 
 inventory.sort(Comparator.comparing(Apple::getWeight)); 
```

# stream流

## 使用流

例子：

```
List<String> names = Arrays.asList("Tom", "John", "Cat");
List<String> collect = names.stream()
        .filter(name -> name.length() < 5)
        .limit(3)
        .collect(Collectors.toList());
```

操作分为：中间操作和终端操作；

```
中间操作  描述
filter     T -> boolean
map   T -> R
sorted  (T, T) -> int
limit  限制个数
skip  跳过，和limit互补
distinct 去重
flatMap  扁平化（下面详解）

终端操作
forEach  遍历并消费流，返回void
count   返回流中元素的个数，返回Long
collect  把流归约成一个集合，比如 List、Map 甚至是 Integer。（详见下一节）
anyMatch 流中是否有一个元素能匹配给定的谓词（流中有一个匹配就可以），返回boolean
allMatch 流中元素是否都能匹配给定的谓词 （都匹配）, 返回boolean
noneMatch 确保流中没有任何元素与给定的谓词匹配 （都不匹配）, 返回boolean
findAny findFirst 查找，用Optional接收（下面详解）
reduce  归约（把一个流中的元素组合起来，下面详解）
```



流的扁平化

```
# 将单词转成字符，并去重
List<String> c = words.stream()
  .map(word -> word.split("")) # 单词分割成字符
     .flatMap(Arrays::stream)  # 将多个流合并成一个流（扁平化）
     .distinct()
     .collect(Collectors.toList());
```



Optional和Find查找元素

```
Optional<Integer> optional = someNumbers.stream().filter(x -> x % 3 == 0).findFirst();
optional代表一个值存在或不存在，这样就不用返回众所周知容易出问题的null了，它还有许多方法，如：
 isPresent()
 ifPresent(Consumer<T> block)会在值存在的时候执行给定的代码块
 orElse(T other) 值存在时返回值，否则返回一个默认值

someNumbers.stream()
 .filter(x -> x % 3 == 0)
 .findAny()
 .ifPresent(x -> System.out.println(x)); // 如果存在就输出
String s = names.stream()
    .filter(name -> name.length() < 5)
    .findFirst().orElse(""); // 如果存在就返回值，否则就返回""
```



reduce归约

```
// 1. 求和
// 如果reduce有参数，则返回结果
int sum = numbers.stream().reduce(0, (a, b) -> a + b); 
// 如果reduce没有参数，返回Optional(因为stream可能为空)
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b)); 
// 2. 最小值
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```



数值流

```
// 每个Integer都必须拆箱成原始类型再进行求和
int calories = menu.stream().map(Dish::getCalories).sum(); 
// IntStream、DoubleStream和LongStream，避免了装箱成本
1. 映射成数值流
int calories = mean.stream().mapToInt(Dish::getCalories).sum();
2. 转换回对象流
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();

// 数值范围：IntStream有静态方法range和rangeClosed生成范围内数值
// rangeClosed(x, y)包括两端数字， range()不包括后端数字
IntStream evenNumbers = IntStream.rangeClosed(1, 100).filter(n -> n % 2 == 0);
```

## 构建流

由值创建

```
Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action"); 
Stream<String> emptyStream = Stream.empty();
```

由数组创建

```
int[] numbers = {2, 3, 5, 7, 11, 13}; 
int sum = Arrays.stream(numbers).sum();
```

由文件生成流

```
long uniqueWords = 0; 
try(Stream<String> lines = Files.lines(Paths.get("data.txt"))){ 
 uniqueWords = lines.map(line -> line.split(""))
      .flatMap(Arrays.stream())
       .distinct().count(); 
} 
catch(IOException e){ 
}
```

由函数生成流：创建无限流 iterate和generate

```
// Stream.iterate 迭代式
Stream.iterate(0, n -> n + 2).limit(10).forEach(System.out::println); 
// Stream.generate 生成
Stream.generate(Math::random).limit(5).forEach(System.out::println); 
```

## collect和Collectors

Collectors类提供了许多方法用于创建收集器，能进一步简化代码，主要提供了三种功能：

* 将流元素归约和汇总为一个值
* 元素分组
* 元素分区（分成true和false两组）

归约/汇总

```
//数量
long howManyDishes = menu.stream().collect(Collectors.counting()); 
//热量和
int totalCalories = menu.stream().collect(Collectors.summingInt(Dish::getCalories)); 
// 平均值
double avgCalories = menu.stream().collect(Collectors.averagingInt(Dish::getCalories)); 
// 字符串拼接
String shortMenu = menu.stream().map(Dish::getName).collect(Collectors.joining(","));
// List
List<String> names = mean.stream().map(Dish::getName).collect(Collectors.toList());
```

分组

```
// 分组 (groupingBy还可以嵌套，变成多级分组)
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(Collectors.groupingBy(Dish::getType));
// 自定义分组
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect( 
 groupingBy(dish -> { 
        if (dish.getCalories() <= 400) return CaloricLevel.DIET; 
       else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL; 
       else return CaloricLevel.FAT;
})); 
```

分区（分成true和false两组）

```
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));
```

## Collector接口

我们模仿`Collectors.toList()`自定义一个方法

```
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> 
```

其中Collector接口定义如下：

```
// T是流中要收集的项目的泛型
// A是累加器的类型，累加器是在收集过程中用于累积部分结果的对象。
// R是收集操作得到的对象（通常但并不一定是集合）的类型。
public interface Collector<T, A, R> { 
    Supplier<A> supplier(); 
    BiConsumer<A, T> accumulator(); 
    Function<A, R> finisher(); 
    BinaryOperator<A> combiner(); 
    Set<Characteristics> characteristics(); 
} 
```

定义第一个函数：建立新的结果容器，supplier方法

> 创建一个空的累加器实例，供数据收集过程使用。

```
public Supplier<List<T>> supplier() {
 return () -> new ArrayList<T>(); 
} 
```

定义第二个函数：将元素添加到结果容器，accumulator方法

>accumulator方法会返回执行归约操作的函数。当遍历到流中第n个元素时，这个函数执行时会有两个参数：保存归约结果的累加器（已收集了流中的前 n-1 个项目），还有第n个元素本身。

```
public BiConsumer<List<T>, T> accumulator() { 
 return (list, item) -> list.add(item); 
} 
```

定义第三个函数： 对结果容器应用最终转换，finisher方法

> 在遍历完流后，finisher方法必须返回在累积过程的最后要调用的一个函数，以便将累加器对象转换为整个集合操作的最终结果。

```
// ToListCollector累加器对象恰好符合预期的最终结果，因此无需进行转换。
public Function<List<T>, List<T>> finisher() { 
 return Function.identity(); 
} 
```

定义第四个函数： 合并两个结果容器：combiner方法

> combiner方法会返回一个供归约操作使用的函数，它定义了对流的各个子部分进行并行处理时，各个子部分归约所得的累加器要如何合并。

```
public BinaryOperator<List<T>> combiner() { 
    return (list1, list2) -> { 
    list1.addAll(list2); 
    return list1; } 
} 
```

定义第五个函数： characteristics方法

>characteristics会返回一个不可变的Characteristics集合，它定义了收集器的行为——尤其是关于流是否可以并行归约，以及可以使用哪些优化的提示。Characteristics是一个包含三个项目的枚举。
>
>* UNORDERED——归约结果不受流中项目的遍历和累积顺序的影响。
>* CONCURRENT——accumulator函数可以从多个线程同时调用，且该收集器可以并行归约流。
>* IDENTITY_FINISH——这表明完成器方法返回的函数是一个恒等函数，可以跳过。

```
public Set<Characteristics> characteristics() { 
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, CONCURRENT));
} 
```

将ToListCollector及其方法定义完，就可以用了

```
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());
```

# stream并行

## 并行流

**并行流**

Java 7引入了一个叫作分支/合并的框架，让并行处理数据变的简单。stream接口允许你声明性地将顺序流变为并行流，其底层也是用的是分支/合并框架。

```
public static long parallelRangedSum(long n) { 
     return LongStream.rangeClosed(1, n)
                      .parallel()
                      .reduce(0L, Long::sum);
} 
```

**正确使用并行流**

下面是另一种实现对前n个自然数求和的方法

```
public static long sideEffectSum(long n) { 
     Accumulator accumulator = new Accumulator(); 
     LongStream.rangeClosed(1, n).forEach(accumulator::add); 
     return accumulator.total; 
} 
public class Accumulator { 
     public long total = 0; 
     public void add(long value) { total += value; } 
} 
```

因为它在本质上就是顺序的。每次访问total都会出现数据竞争。如果你尝试用同步来修复，那就出现错误：

> 共享可变状态会影响并行流以及并行计算。（在段代码就是total就是共享可变状态）

```
public static long sideEffectParallelSum(long n) { 
     Accumulator accumulator = new Accumulator(); 
     LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add); 
     return accumulator.total; 
} 
```

**高效使用并行流**

* 并行流并不总是比顺序流快
* 留意装箱，或者用原始数据流（IntStream LongStream DoubleStream）
* limit和findFirst等依赖于元素顺序的操作，它们在并行流上执行的代价非常大。
* 对于较小的数据量，选择并行流几乎从来都不是一个好的决定。
* 要考虑流背后的数据结构是否易于分解。例如，ArrayList的拆分效率比LinkedList高得多
* 还要考虑终端操作中合并步骤的代价是大是小

## 分支/合并框架

> 分支/合并框架就是：定义RecursiveTask，并实现它唯一的抽象方法compute

用分支/合并框架执行并行求和

```
public class ForkJoinSumCalculator extends java.util.concurrent.RecursiveTask<Long> { 
     private final long[] numbers; 
     private final int start;
     private final int end; 
     public static final long THRESHOLD = 10_000; 
     // 公共构造函数，供外部调用
     public ForkJoinSumCalculator(long[] numbers) { 
      this(numbers, 0, numbers.length); 
     }
     // 私有构造函数，以便于调用子任务
     private ForkJoinSumCalculator(long[] numbers, int start, int end) { 
         this.numbers = numbers; 
         this.start = start; 
         this.end = end;
     } 
     @Override 
     protected Long compute() { 
        int length = end - start; 
        if (length <= THRESHOLD) { 
           return computeSequentially(); 
        } 
        // 异步执行LeftTask
      ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2); 
      leftTask.fork();
      // 同步执行RightTask
      ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end); 
      Long rightResult = rightTask.compute(); 
      // 读取LeftTask子任务结果，如果尚未完成就等待
      Long leftResult = leftTask.join(); 
      // 返回两个结果
      return leftResult + rightResult;
     } 
     // 最小任务顺序计算
     private long computeSequentially() { 
         long sum = 0;
         for (int i = start; i < end; i++) {{ 
          sum += numbers[i];
         } 
         return sum; 
     } 
}
```

调用：

```
public static long forkJoinSum(long n) { 
     long[] numbers = LongStream.rangeClosed(1, n).toArray(); 
     ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers); 
     return new ForkJoinPool().invoke(task); 
} 
```

注意点：

* ForkJoinPool使用Runtime.availableProcessors的返回值来决定线程池使用的线程数。

## Spliterator

Spliterator是Java 8中加入的另一个新接口；和Iterator一样，Spliterator也用于遍历数据源中的元素，但它是为了并行执行而设计的。Java 8已经为集合框架中包含的所有数据结构提供了一个默认的Spliterator实现。集合实现了Spliterator接口，接口提供了一个pliterator方法。这个接口定义了若干方法，如下面的代码清单所示。

```
public interface Spliterator<T> { 
     boolean tryAdvance(Consumer<? super T> action);   // 和Iterator一样
     Spliterator<T> trySplit();        // 用于并行
     long estimateSize();          // 估计还剩下多少元素要遍历
     int characteristics(); 
} 
```

* stream流在并行处理时，不断调用trySplit()方法，直到所有的trySplit()返回null
* spliterator也可以自己创建（略）