### lambda表达式

体验开启一个线程

可以看到返回表达式返回值是一个Runnable接口，接口实例作为Thread的入参。

```
new Thread(() -> System.out.println("lambda开启线程   " +"线程名："+Thread.currentThread().getName()),
        "lambda线程名").start();
        
        //结果  lambda开启线程   线程名：lambda线程名
```

- -> 左边是函数入参
- ->右边是函数执行体，也是方法的内部逻辑实现
- 只适用于只有一个方法的接口

```
   @FunctionalInterface
   public interface LambdaDemo1 {
     int doubleNum(int i);
     
     default int add(int x,int y){
        return x+y;
    }
   public class Test {
        public static void main(String[] args) {
            //获得接口实例
            LambdaDemo1 lambdaDemo1 = (i) -> i*2;
            LambdaDemo1 lambdaDemo2 = (i) -> i*4;
            //操作接口实例的default修饰的方法 
            System.out.println( lambdaDemo1.add(1,2));
            //操作接口实例的方法 返回40 返回值由lambda表达式函数体逻辑确定
            System.out.println(lambdaDemo1.doubleNum(20));
            //操作接口实例的方法 返回80
            System.out.println(lambdaDemo2.doubleNum(20));
        }
    }
}
```

### default 和static

default 和static修饰的接口方法子类可以不继承 可以参考springboot的 WebMvcConfig类

default和static 修饰的接口方法要默认实现

接口的子类可以直接调用接口中default修饰的方法

通过类名.方法名可以直接调用方法

### 函数接口

JDK 8 中提供了一组常用的核心函数接口：

| 接口              | 参数   | 返回类型 | 描述                                                         |
| :---------------- | :----- | :------- | :----------------------------------------------------------- |
| Predicate<T>      | T      | boolean  | 用于判别一个对象。比如求一个人是否为男性                     |
| Consumer<T>       | T      | void     | 用于接收一个对象进行处理但没有返回，比如接收一个人并打印他的名字 |
| Function<T, R>    | T      | R        | 转换一个对象为不同类型的对象                                 |
| Supplier<T>       | None   | T        | 提供一个对象                                                 |
| UnaryOperator<T>  | T      | T        | 接收对象并返回同类型的对象                                   |
| BinaryOperator<T> | (T, T) | T        | 接收两个同类型的对象，并返回一个原类型对象                   |

`其中 `Cosumer` 与 `Supplier对应，一个是消费者，一个是提供者。

```
//消费函数 //消费字符串打印出来
Consumer<String> consumer = s -> System.out.println(s + "lambda表达式函数实现体");
consumer.accept("输入的数据s  ");
```

Predicate 用于判断对象是否符合某个条件，经常被用来过滤对象。

```
        //断言函数 判断传如的参数是否大于0
        //i是传入的参数  i>0是函的操作
        Predicate<Integer> pridicate = i -> i > 0;
        //与上面相比不用手写泛型
        IntPredicate predicate1 = i -> i > 0;
        //test方法返回为boolean 可以看jdk源码
        System.out.println(pridicate.test(-9));
```

Function` 是将一个对象转换为另一个对象，比如说要装箱或者拆箱某个对象。`

UnaryOperator` 接收和返回同类型对象，一般用于对对象修改属性。`BinaryOperator 则可以理解为合并对象。

### 方法引用

**方法引用**是用来直接访问类或者实例的已经存在的方法。方法引用提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文。计算时，方法引用会创建函数式接口的一个实例。

下面定义一个类中两个方法一个静态方法，一个非静态方法 。                                                     

```
class Dog {
    private String name = "哮天犬";

    private int food = 10;

    public static void bark(Dog dog) {
        System.out.println(dog + "叫了");
    }

    public int eat(int num) {
        System.out.println("吃了 " + num + "斤狗粮");
        this.food -= num;
        return this.food;
    }
}
```

- 静态方法引用

**类名：：方法名**

```
//静态方法的方法引用
//消费函数引用Dog的bark方法
Consumer<Dog> consumer1 = Dog::bark;
Dog dog = new Dog();
consumer1.accept(dog);
```

- 非静态方法引用

**实例对象名：：方法**

```
//非静态方法的方法引用
Function<Integer, Integer> function = dog::eat;
//两个参数一样 可以用下面这种
UnaryOperator<Integer> function1 = dog::eat;
System.out.println("还剩下" + function.apply(2) + "斤");
```

### **Stream** 

参考文章 https://www.jianshu.com/p/a6ee65618a1c

- Stream 不是数据结构，不保存数据，它是有关算法和计算的，就如同一个高级版本的迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。同时又与迭代器不同，迭代器只能串行操作，Stream可以并行化操作。

- 当我们使用一个流的时候，通常包括三个基本步骤：

获取一个数据源（source)→数据转换→执行操作获取想要的结果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道

- 外部迭代

最传统的方法是用Iterator，当然还以用for i、增强for循环等等。这一类方法叫做**外部迭代**，意为显式地进行迭代操作，即集合中的元素访问是由一个处于集合外部的东西来控制的，在这里控制着循环的东西就是迭代器。

```
int[] nums = {1, 2, 3};
int sum = 0;
for (int i : nums) {
    sum += i;
}
System.out.println("结果为："+sum);
```

- 内部迭代

```
int sum1 = IntStream.of(nums).sum();
System.out.println("结果为："+sum1);
```

### 流的使用

常见操作

|          |          |                                                              |
| -------: | :------: | :----------------------------------------------------------- |
| 中间操作 |  无状态  | map (mapToInt, flatMap 等)、filter、peek                     |
|          |  有状态  | distinct、sorted、limit、skip                                |
| 终结操作 |  非短路  | forEach、forEachOrdered、toArray、reduce、collect、min、 max、 count |
|          | 短路操作 | anyMatch、allMatch、noneMatch、findFirst、findAny            |

Stream中的操作可以分为两大类：**中间操作与终结操作**

- 中间操作（Intermediate）：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。中间操作又可以分为无状态（Stateless）操作与有状态（Stateful）操作，前者是指元素的处理不受之前元素的影响；后者是指该操作只有拿到所有元素之后才能继续下去。
- 终结操作（Terminal）：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果。终结操作又可以分为短路与非短路操作，短路是指遇到某些符合条件的元素就可以得到最终结果,比如找到第一个满足条件的元素。而非短路是指必须处理所有元素才能得到最终结果。

### 流的中间操作

- #### map

  对流中的每个元素执行一个函数，使得元素转换成另一种类型输出。

  入参 Function<T, R>

  ```
  String str = "my name is 007";
  //打印出单词长度大于2的单词的长度
  Stream.of(str.split(" ")).filter(s -> s.length() > 2).map(s -> s.length()).forEach(System.out::println);
  ```

  

- ####  flatMap

入参 Function<T, ? extends Stream>

转化出的还是流

```
//flatMap 的使用
//intStream/longStream不是Stream的子类，所以要装箱 boxed（）
Stream.of(str.split(" ")).flatMap(s -> s.chars().boxed()).forEach(
        i -> System.out.println((char) i.intValue()));
```

- #### filter

filter 对原始 Stream 进行某项测试，通过测试的元素被留下来生成一个新 Stream。 （入参 Predicate）数字要大于100 小于1000，filter就是过滤。

```
//创建一个无限流  limit可以指定大小限定打印出10个符合条件的元素
new Random().ints().filter(i->i>100&&i<1000).limit(10).forEach(System.out::println);
```



- #### peek

peek 方法我们可以拿到元素，然后做一些其他事情。（入参 Consumer）

把流打印两次

```
Stream.of(str.split(" ")).peek(System.out::println).forEach(System.out::println);
```



- #### limit/skip

limit 返回 Stream 的前面 n 个元素，skip 则是扔掉前 n 个元素

创建一个无限流  limit可以指定大小限定打印出10个符合条件的元素

```
//创建一个无限流  limit可以指定大小限定打印出10个符合条件的元素
new Random().ints().filter(i->i>100&&i<1000).limit(10).forEach(System.out::println);
```

- #### sorted

对元素进行排序 （入参 Comparator）

- #### forEach

对元素进行遍历消费 （入参 Consumer）



 