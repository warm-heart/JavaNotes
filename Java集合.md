Java集合大致可以分为Set、List、Queue、Map四种体系。

 Set：代表无序、不可重复 （TreeSet和LinkedHashSet有序）

List：代表有序、重复的集合

 Queue：代表一种队列集合实现 

Map：代表具有映射关系的集合

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

线程不安全：

两个线程同时添加元素，得到size=1，线程1在下标1处添加元素，线程1挂起，线程2也在下标1处添加并修改size为2，线程1重新运行，size变为3，造成元素丢失。

数组越界

**列表大小为9，即size=9**
线程A开始进入add方法，这时它获取到size的值为9，调用ensureCapacityInternal方法进行容量判断。
线程B此时也进入add方法，它获取到size的值也为9，也开始调用ensureCapacityInternal方法。
线程A发现需求大小为10，而elementData的大小就为10，可以容纳。于是它不再扩容，返回。
线程B也发现需求大小为10，也可以容纳，返回。
线程A开始进行设置值操作， elementData[size++] = e 操作。此时size变为10。
线程B也开始进行设置值操作，它尝试设置elementData[10] = e，而elementData没有进行过扩容，它的下标最大为9。于是此时会报出一个数组越界的异常ArrayIndexOutOfBoundsException.

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

### HashMap 遍历

keySet 其实是遍历了 2 次，一次是转为 Iterator 对象，另一次是从 hashMap 中取出 key 所对应的 value。而 entrySet 只是遍历了一次就把 key 和 value 都放到了 entry 中，效 率更高。如果是 JDK8，使用 Map.foreach 方法

1. entrySet（alibaba开发手册建议遍历）

   entrySet()的返回值也是返回一个Set集合，此集合的类型为Map.Entry 表示key-value

   ```
    for (Map.Entry entry : hashMap.entrySet()) {
               System.out.println(entry.getKey() + "=" + entry.getValue());
           }
   ```

2. keySet方式遍历

   keySet()方法返回值是Map中key值的集合

   ```
    Iterator iterator = hashMap.keySet().iterator();
           while (iterator.hasNext()) {
               Object key = iterator.next();
               System.out.println("key是： " + key + " value是： " +                   hashMap.get(key));
   
           }
   ```

   iterator.next()方法获取的是map中的key，hashMap.get(key)是获取key对应的value

   

3. iterator entrySet方式遍历

   ```
    Iterator iterator1 = hashMap.entrySet().iterator();
           while (iterator1.hasNext()) {
               Map.Entry entry = (Map.Entry) iterator1.next();
               System.out.println("key是： " + entry.getKey() + " value是： " + entry.getValue());
           }
   ```

   

4. jdk8 HashMap.foreach()遍历

   ```
    hashMap.forEach((k,v)->{
              System.out.println(v);
          });
   ```

### ConCurrentHashmap

使用CAS与在操作hash值相同的链表的头结点还是会synchronized上锁，这样才能保证线程安全。

```
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());//计算hash值，两次hash操作
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {//类似于while(true)，死循环，直到插入成功 
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)//检查是否初始化了，如果没有，则初始化
                tab = initTable();
                /*
                    i=(n-1)&hash 等价于i=hash%n(前提是n为2的幂次方).即取出table中位置的节点用f表示。
                    有如下两种情况：
                    1、如果table[i]==null(即该位置的节点为空，没有发生碰撞)，则利用CAS操作直接存储在该位置，
                        如果CAS操作成功则退出死循环。
                    2、如果table[i]!=null(即该位置已经有其它节点，发生碰撞)
                */
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)//检查table[i]的节点的hash是否等于MOVED，如果等于，则检测到正在扩容，则帮助其扩容
                tab = helpTransfer(tab, f);//帮助其扩容
            else {//运行到这里，说明table[i]的节点的hash值不等于MOVED。
                V oldVal = null;
                synchronized (f) {//锁定,（hash值相同的链表的头节点）
                    if (tabAt(tab, i) == f) {//避免多线程，需要重新检查
                        if (fh >= 0) {//链表节点
                            binCount = 1;
                            /*
                            下面的代码就是先查找链表中是否出现了此key，如果出现，则更新value，并跳出循环，
                            否则将节点加入到里阿尼报末尾并跳出循环
                            */
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)//仅putIfAbsent()方法中onlyIfAbsent为true
                                        e.val = value;//putIfAbsent()包含key则返回get，否则put并返回  
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {//插入到链表末尾并跳出循环
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) { //树节点，
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {//插入到树中
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //插入成功后，如果插入的是链表节点，则要判断下该桶位是否要转化为树
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)//实则是>8,执行else,说明该桶位本就有Node
                        treeifyBin(tab, i);//若length<64,直接tryPresize,两倍table.length;不转树 
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

**思路：**

1、检查key/value是否为空，如果为空，则抛异常，否则进行2
2、进入for死循环，进行3
3、检查table是否初始化了，如果没有，则调用initTable()进行初始化然后进行 2，否则进行4
4、根据key的hash值计算出其应该在table中储存的位置i，取出table[i]的节点用f表示。
    根据f的不同有如下三种情况：

1）如果table[i]==null(即该位置的节点为空，没有发生碰撞)， 则利用CAS操作直接存储在该位置，如果CAS操作成功则退出死循环。

 2）如果table[i]!=null(即该位置已经有其它节点，发生碰撞)，碰撞处理也有两种情况                       

2.1）检查table[i]的节点的hash是否等于MOVED，如果等于，则检测到正在扩容，则帮助其扩容                 

2.2）说明table[i]的节点的hash值不等于MOVED，如果table[i]为链表节点，则将此节点插入链表中即可， 如果table[i]为树

节点，则将此节点插入树中即可。插入成功后，进行 5

5、如果table[i]的节点是链表节点，则检查table的第i个位置的链表是否需要转化为数，如果需要则调用treeifyBin函数进行转化
、