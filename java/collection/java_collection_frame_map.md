> 我们是使用数组来保存数据，但是他的长度一旦创建，就已经确定了，当我们要动态传入穿值，数组就有些局限了，集合类就孕育而生；
所谓集合，就是来保存，盛装数据，也可称为容器类；使用的类在java.util包里。

## 集合类的分类

- Collection（一组对立的元素）
  - List（有顺序）
  - Set（不能有重复元素）
  - Queue（保存队列先进先出的顺序）
- Map（键值对（key-value））

## Collection & Map 区别

- Collection 每个位置只能保存一个元素
- Map 保存的是键值对（key-value），可以通过过key，来找到value

## Java 集合类的层次

![img](/imgs/java_collection_frame_map.png)


### 层次关系

#### Interface Iterable

Collection 的父接口，可以使用foreach进行遍历，还有迭代器iterator()方法进行遍历

#### Collection

最基本的集合接口，代表一组Object 集合

#### Set

实现Collectiion 接口，不能包含重复的元素
判断两个对象是否相同：equals()方法，新加入的元素和已有元素相比equals 都返回false，否则拒绝添加
需要注意的是，Set 底层实现都是Map，Map 的value 为空；

#### HashSet

使用Hash 算法来存储元素，具有良好的存取和查找性能；
然后根据元素的Hashcode 值来决定元素在Hashset 的为位置（会重新进行排序）
Hashset 判断两个元素是否相同：
- equals 为true
- hashcode 相同

#### LinkedHashSet

和HashSet 一样，是利用hashCode 来觉得元素的存储位置，但是使用链表维护元素的次序
当遍历的时候，是按照插入的顺序遍历的
LinkedHashSet 需要维护元素的插入顺序，因此性能会略低于HashSet的性能，但是在迭代访问所有元素是性能会很高（链表适合遍历）

#### SortedSet

排序，compareTo() 方法进行比较，插入的元素类型需要一样，否则出现ClassCastException

#### TreeSet

SortedSet 的实现类，保证元素处于排序状态

#### EnumSet

为枚举类设计的集合类

#### List

有顺序，可重复，因此可以通过索引来指定位置的集合元素

#### ArrayList

基于数组的List，封装了动态增长的Object[] 数组

#### Vector

和ArrayList 几乎一样，比较古老，线程安全

#### Stack

是Vector 的子类，栈的结构（后进先出）

#### LinkedList

实现List，Deque；实现List，可以进行队列操作，可以通过索引来随机访问集合元素；实现Deque，也可当作双端队列，也可当作栈来使用

#### Queue（队列）

Queue 内部是队列的数据结构（先进先出），新插入的元素会在尾部；插入之后，会慢慢向顶部移动；
类似生活中的排队

#### PriorityQueue（优先级队列)

除了实现Queue 接口，PriorityQueue 还对插入的元素进行重新排序（Comparator）

#### Deque

双端队列，Deque 可以从两端来添加啊，删除元素，因此，Deque 可以当作队列使用，也可当作栈来使用

#### ArrayDeque

基于数组的双端队列，和ArrayList 类似，底层都是采用一个动态可分配的Object[] 的数组

#### LinkedList

### Map

保存映射关系的数据（key-value）
key：不允许重复，即两个任意的key 通过equals 比较返回总是false
value：存储方式类似List（可以重复，根据索引来查找）
需要注意的是：Java 中先实现Map，然后通过包装所有value 为null 来实现Set 集合

#### HashMap

和Hashset 一样，利用key 的hashCode 值进行排序；
判断两个key 相等，equals() 方法返回true，两个hashCode 也必须相等

#### LinkedHashMap

使用双向链表来维护key-value 的次序，该链表负责维护Map 的迭代顺序，与插入时的顺序一样

#### HashTable

古老的Map 实现类

#### Properies

将Map 写入文件

#### SortedMap

#### TreeMap

是一个红黑树结构，对key 进行排序（compareTo），
自然排序：默认的排序
定制排序：也可在构造方法中传入Comparetor 对象，自定义排序

#### WeakHashMap

"弱引用"，可能会被垃圾回收

#### IdentityHashMap

key 的比对 "=="

#### EnumMap

枚举类实现的Map

### 遍历的几种方式

- iterator
- foreach
- listIterator（List 特有的）

## 总结

### Set 的选择

- 1. HashSet的性能总是比TreeSet好(特别是最常用的添加、查询元素等操作)，因为TreeSet需要额外的红黑树算法来维护集合元素的次序。只有当需要一个保持排序的Set时，才应该使用TreeSet，否则都应该使用HashSet
- 2. 对于普通的插入、删除操作，LinkedHashSet比HashSet要略慢一点，这是由维护链表所带来的开销造成的。不过，因为有了链表的存在，遍历LinkedHashSet会更快
- 3. EnumSet是所有Set实现类中性能最好的，但它只能保存同一个枚举类的枚举值作为集合元素
- 4. HashSet、TreeSet、EnumSet都是"线程不安全"的，通常可以通过Collections工具类的synchronizedSortedSet方法来"包装"该Set集合。SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));

### List 选择

- 1. java提供的List就是一个"线性表接口"，ArrayList(基于数组的线性表)、LinkedList(基于链的线性表)是线性表的两种典型实现
- 2. Queue代表了队列，Deque代表了双端队列(既可以作为队列使用、也可以作为栈使用)
- 3. 因为数组以一块连续内存来保存所有的数组元素，所以数组在随机访问时性能最好。所以的内部以数组作为底层实现的集合在随机访问时性能最好。
- 4. 内部以链表作为底层实现的集合在执行插入、删除操作时有很好的性能
- 5. 进行迭代操作时，以链表作为底层实现的集合比以数组作为底层实现的集合性能好
- 6. 当要大量的插入，删除，应当选用LinkedList；当需要快速随机访问则选用ArrayList;

### Map 的选择

- 1. HashMap和Hashtable的效率大致相同，因为它们的实现机制几乎完全一样。但HashMap通常比Hashtable要快一点，因为Hashtable需要额外的线程同步控制
- 2. TreeMap通常比HashMap、Hashtable要慢(尤其是在插入、删除key-value对时更慢)，因为TreeMap底层采用红黑树来管理key-value对
- 3. 使用TreeMap的一个好处就是： TreeMap中的key-value对总是处于有序状态，无须专门进行排序操作
- 4. HahMap 是利用hashCode 进行查找，而TreeMap 是保持者某种有序状态
- 5. 所以，插入，删除，定位操作时，HashMap 是最好的选择；如果要按照自然排序或者自定义排序，那么就选择TreeMap

