Java集合大致可以分为Set、List、Queue、Map四种体系。

 Set：代表无序、不可重复 List：代表有序、重复的集合 Queue：代表一种队列集合实现 Map：代表具有映射关系的集合



# Set

Set接口扩展于Collection接口,在一个实现Set的类中必须确保该规则集没有相同的元素。Set接口中有三个具体类：散列集HashSet、链式散列集LinkedHashSet和树形集TreeSet。

### HashSet

HashSet扩展于Set接口，可以用来储存互不相同的元素。当程序向HashSet的实例中添加多个相同的元素时，只有一个元素会被存储，因为规则集中只能存储不同的元素。此外，HashSet实例中存储的元素没有特定的顺序，并不会按照插入顺序进行排序。

### TreeSet

TreeSet是SortSet接口中的一个具体子类，其中SortSet为Set的子接口。在LinkedHashSet中，可以通过元素插入的顺序对元素排序，但是有时候需要自定义元素排序的顺序，在TreeSet中，只要对象可比较，即可添加进树形集中，并且可通过以下两种方式进行排序：

1.使用Comparable接口实现。当插入的对象为Comparable实例（如String,Date等）时，就可以通过接口中的compareTo对对象进行排序。此种排序方式为自然顺序。

2.使用比较器接口Comparator实现。有时我们可能需要自定义元素排序的顺序，或者说对象不是Comparable的实例，就可以通过比较器中的compare(object e1, object e2)方法来实现自定义的排序。此种排序方式为比较器顺序



# List

List代表是一个元素有序、可重复的集合，集合中每个元素都有对应的顺序索引。List集合允许使用重复的元素，可以通过索引来访问指定位置的元素。List集合默认按元素的添加顺序来设置元素的索引，例如第一次添加的元素索引为0，第二次添加的元素索引为0，依次类推下去

### ArrayList

在Java2之前引入了向量类Vector，其使用方式与ArrayList类似，但Vector实现了现成同步，以避免多线程访问数据时引起数组损坏

LinkedList



# Queue

Queue，队列，是一种先进先出的数据结构。新增的元素会插在队列的末尾。在优先队列中，优先级高的元素会首先出队。

Queue中有两个具体实现类：链表LinkedList和优先队列PriorityQueue。

### LinkedList

在上述的List接口中也提到过LinkedList，它同时扩展自List接口和Deque接口。双向队列Deque接口扩展自Queue接口，支持在队列的两端在两端插入或删除数据。具体方法可参考上述内容。

### PriorityQueue

 此类实现了优先队列，在默认情况下，该队列的初始容量为11。其实例所存储的元素默认以自然顺序排列，因此自然顺序下最小的元素会优先出队。队列中可能出现对个优先级相同的元素，那么拥有相同优先级的元素会有其中任意一个优先出队。在讲述TreeSet时提到过使用Comparator接口来实现比较器顺序，在优先队列中依然可行。





# Map

### 与Set集合的关系

如果 把Map里的所有key放在一起看，它们就组成了一个Set集合（所有的key没有顺序，key与key之间不能重复），实际上Map确实包含了一个keySet()方法，用户返回Map里所有key组成的Set集合。

 ### 与List集合的关系

如果把Map里的所有value放在一起来看，它们又非常类似于一个List：元素与元素之间可以重复，每个元素可以根据索引来查找，只是Map中索引不再使用整数值，而是以另外一个对象作为索引。

