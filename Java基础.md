# ==和equals

### 判空防止空指针

判空操作时 常量最好写在前面 不然会出现空指针异常

```
if（s==null||"".equals（s））
```



### ==和equals的区别

== 比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。比较的是真正意义上的指针操作。

基本类型  ==是判断值是否相等

引用类型 ==是判断对象是否相等

equals用来比较的是两个对象的内容是否相等，由于所有的类都是继承自java.lang.Object类的，所以适用于所有对象，如果没有对该方法进行覆盖的话，调用的仍然是Object类中的方法，而Object中的equals方法返回的却是==的判断。String类重写了equals方法

```
 //引用类型
       List list = new ArrayList();
       List list1 = new ArrayList();
       System.out.println(list==list1);   //false
       System.out.println(list.equals(list1));  //true
```

### 重写equals方法和hashCode方法

```
public class User {

    private String name;
    // static String age ="18";
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

}
```

当用user类作为hashMap的key时必须重写equals和hashCode方法

```
 @Override
    public int hashCode() {
        int B = 31;
        int hash = 0;
        hash = hash * B + name.hashCode();
        hash = hash * B + age;
        return hash;
    }
```

```
@Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        User another = (User) obj;
        return this.name.equals(another.name) && this.age == another.age;
    }
```

# 异常



### Java异常的分类

- Java标准库内建了一些通用的异常，这些类以Throwable为顶层父类。
- Throwable又派生出Error类和Exception类。
- Error类以及他的子类的实例，代表了JVM本身的错误。错误不能被程序员通过代码处理，Error很少出现。
- 程序员应该关注Exception为父类的分支下的各种异常类。

### 异常处理的基本语法

try…catch…finally语句块

```
try{
     //try块中放可能发生异常的代码。
     //如果执行完try且不发生异常，则接着去执行finally块和finally后面的代码（如果有的话）。
     //如果发生异常，则尝试去匹配catch块。
 
}catch(SQLException SQLexception){
    //每一个catch块用于捕获并处理一个特定的异常，或者这异常类型的子类。Java7中可以将多个异常声明在一个catch中。
    //catch后面的括号定义了异常类型和异常参数。如果异常与之匹配且是最先匹配到的，则虚拟机将使用这个catch块来处理异常。
    //在catch块中可以使用这个块的异常参数来获取异常的相关信息。异常参数是这个catch块中的局部变量，其它块不能访问。
    //如果当前try块中发生的异常在后续的所有catch中都没捕获到，则先去执行finally，然后到这个函数的外部caller中去匹配异常处理器。
    //如果try中没有发生异常，则所有的catch块将被忽略。
 
}catch(Exception exception){
    //...
}finally{
 
    //finally块通常是可选的。
   //无论异常是否发生，异常是否匹配被处理，finally都会执行。
   //一个try至少要有一个catch块，否则， 至少要有1个finally块。但是finally不是用来处理异常的，finally不会捕获异常。
  //finally主要做一些清理工作，如流的关闭，数据库连接的关闭等。 
}
```

1. try块中的局部变量和catch块中的局部变量（包括异常变量），以及finally中的局部变量，他们之间不可共享使用。
2. 每一个catch块用于处理一个异常。异常匹配是按照catch块的顺序从上往下寻找的，只有第一个匹配的catch会得到执行。匹配时，不仅运行精确匹配，也支持父类匹配，因此，如果同一个try块下的多个catch异常类型有父子关系，应该将子类异常放在前面，父类异常放在后面，这样保证每个catch块都有存在的意义。
3. 、java中，异常处理的任务就是将执行控制流从异常发生的地方转移到能够处理这种异常的地方去。也就是说：当一个函数的某条语句发生异常时，这条语句的后面的语句不会再执行，它失去了焦点。执行流跳转到最近的匹配的异常处理catch代码块去执行，异常被处理完后，执行流会接着在“处理了这个异常的catch代码块”后面接着执行。
   有的编程语言当异常被处理后，控制流会恢复到异常抛出点接着执行，这种策略叫做：resumption model of exception handling（恢复式异常处理模式 ）
   而Java则是让执行流恢复到处理了异常的catch块后接着执行，这种策略叫做：termination model of exception handling（终结式异常处理模式）
4. finally块不管异常是否发生，只要对应的try执行了，则它一定也执行。只有一种方法让finally块不执行：System.exit()。因此finally块通常用来做资源释放操作：关闭文件，关闭数据库连接等等。
5. 只要try执行了，finally一定会执行。（在try语句块之前return 不会进入try语句，finally不会执行）

# 泛型

### 泛型好处

- 代码更加简洁【不用强制转换】
- 程序更加健壮【只要编译时期没有警告，那么运行时期就不会出现ClassCastException异常】
- 可读性和稳定性【在编写集合的时候，就限定了类型】

**1，类型安全。** 泛型的主要目标是提高 Java 程序的类型安全。通过知道使用泛型定义的变量的类型限制，编译器可以在一个高得多的程度上验证类型假设。没有泛型，这些假设就只存在于程序员的头脑中（或者如果幸运的话，还存在于代码注释中）。

**2，消除强制类型转换**。 泛型的一个附带好处是，消除源代码中的许多强制类型转换。这使得代码更加可读，并且减少了出错机会

**3，潜在的性能收益。** 泛型为较大的优化带来可能。在泛型的初始实现中，编译器将强制类型转换（没有泛型的话，程序员会指定这些强制类型转换）插入生成的字节码中。但是更多类型信息可用于编译器这一事实，为未来版本的 JVM 的优化带来可能。由于泛型的实现方式，支持泛型（几乎）不需要 JVM 或类文件更改。所有工作都在编译器中完成，编译器生成类似于没有泛型（和强制类型转换）时所写的代码，只是更能确保类型安全而已。

### 泛型的实现原理

泛型的实现是靠类型擦除技术 类型擦除是在编译期完成的 也就是在编译期 编译器会将泛型的类型参数都擦除成它的限定类型，如果没有则擦除为object类型之后在获取的时候再强制类型转换为对应的类型。 在运行期间并没有泛型的任何信息，因此也没有优化。

Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为类型擦除。如在代码中定义的List<object>和List<String>等类型，在编译后都会编程List。JVM看到的只是List，而由泛型附加的类型信息对JVM来说是不可见的。Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法避免在运行时刻出现类型转换异常的情况。类型擦除也是Java的泛型实现方法与C++模版机制实现方式（后面介绍）之间的重要区别。

# 重写与重载

从字面上看，**重写就是 重新写一遍的意思**。其实就是在**子类中把父类本身有的方法重新写一遍**。子类继承了父类原有的方法，但有时子类并不想原封不动的继承父类中的某个方法，所以在方法名，参数列表，返回类型(除过子类中方法的返回值是父类中方法返回值的子类时)都相同的情况下， 对方法体进行修改或重写，这就是重写。但要注意子类函数的访问修饰权限不能小于父类的。

**在同一个类中**，同名的方法如果有不同的参数列表（**参数类型不同、参数个数不同甚至是参数顺序不同**）则视为重载。同时，重载对返回类型没有要求，可以相同也可以不同，但**不能通过返回类型是否相同来判断重载**（返回类型可以相同也可以不同，不是判断是否重载的条件）。 

# 抽象类与接口区别

接口是公开的，不能有私有的方法或变量，接口中的所有方法都`没有方法体`，通过关键字`interface`实现。

抽象类是可以有私有方法或私有变量的，通过把类或者类中的方法声明为abstract来表示一个类是抽象类，被声明为抽象的方法不能包含方法体。子类实现方法必须含有相同的或者更高的访问级别(public->protected->private)。抽象类的子类为父类中所有抽象方法的具体实现，否则也是抽象类。

# 克隆

在实际编程过程中，我们常常要遇到这种情况：有一个对象A，在某一时刻A中已经包含了一些有效值，此时可能 会需要一个和A完全相同新对象B，并且此后对B任何改动都不会影响到A中的值，也就是说，A与B是两个独立的对象，但B的初始值是由A对象确定的。在 Java语言中，用简单的赋值语句是不能满足这种需求的。要满足这种需求虽然有很多途径，但实现clone（）方法是其中最简单，也是最高效的手段。 
Java的所有类都默认继承java.lang.Object类，在java.lang.Object类中有一个方法clone()。JDK API的说明文档解释这个方法将返回Object对象的一个拷贝。要说明的有两点：一是拷贝对象返回的是一个新对象，而不是一个引用。二是拷贝对象与用 new操作符返回的新对象的区别就是这个拷贝已经包含了一些原来对象的信息，而不是对象的初始信息。 

### 深拷贝与浅拷贝

对于引用类型来说，浅拷贝用的还是同一个对象所在的内存地址，如果克隆后的独享修改了属性，那么持有这个引用的所有对象的属性都会被修改，深拷贝则会解决这个问题

### 实现深拷贝方式

##### 所有的引用类型都实现clone方法

```
public class Animal implements Cloneable {
    private String type;
    private Integer age;
    private AnimalHome home;
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        Animal animal = (Animal) super.clone();
        animal.home = (AnimalHome) home.clone();
        return animal;
    }
    
    
    public class AnimalHome implements Cloneable {
    private String locate;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
```

##### 使用序列化

序列化是把对象写到流中便于传输，而反序列化则是把对象从流中读取出来，这里写到流中的对象则是原始对象的一个拷贝，因为原始对象还存在JVM中，所以可以利用序列化产生克隆对象，然后通过反序列化获取这个对象

```
public Object deepClone() throws IOException, ClassNotFoundException {
        //序列化
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);
        //反序列化
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return ois.readObject();
    }
```















