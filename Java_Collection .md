

# java 集合

![](C:\Users\Surpirse\AppData\Roaming\Typora\typora-user-images\Collection)
![image-20221022233806440](C:\Users\Surpirse\AppData\Roaming\Typora\typora-user-images\image-20221022233806440.png)



## Collection接口实现类

特点：

		1. 分为有序、无序、可重复、不可重复数据结构。
		1. 可以使用增强for、iterator迭代器进行遍历。其中List接口实现类可使用索引for遍历
		1. 注意增强for本质是简易的iterator迭代器





### List(ArrayList、Vector、LinkedList)



特点：

		1. 该接口实现类均有序，且可以存放相同、null元素





#### ArrayList

1. ArrayList本质上由object[] elemData进行数据存储。
2. 无参构造器：使用初始elemData数据。
3. 有参构造器(输入初始数组大小): 对 elemData指定大小赋值



==扩容机制==： 若使用无参构造器，第一次添加元素时初始化elemData为长度为10的数组。以后按1.5倍扩容

若使用有参构造器，初始化elemData为指定长度数组，并以该长度进行1.5倍扩容



==可存放元素==: 任何元素、且可重复、有序



__特点__

1. ArrayList*函数线程不安全*，但效率高
2. 扩容按1.5倍扩容
3. 本质上由数组存放元素
4. 增删元素效率较低，需要移动大量元素
5. 查改效率较高，通过索引即可获得对应位置元素





#### LinkedList

__特点__:

1. 本质是双向链表，该类具有first、last字段分别指向链表首节点和尾节点
2. LinkedList为线程不按全类，
3. 增删元素效率较高
4. 查改效率较低
5. 元素有序，且可重复，可放入多个null
6. 线程不安全，没有实现线程同步





#### Vector

*Vector类基本与ArrayList*相同

特点：

1. 线程安全，但效率较ArrayList低。

2. 本质由Object[] 数组存储数据

3. 可使用__无参构造器__直接初始化长度为10的数组。也可使用__有参构造器__指定大小数组

4. ==扩容机制==：

   若使用无参，则基于数组大小10的基础上__==2==__倍扩容；
   若指定初始数组大小，则在指定大小基础上__==2==__二倍扩容。

5. 元素有序，且可重复，可放入多个null







### Set( HashSet、LinkedHashSet、TreeSet )

1. 本质上均为*无序*
2. 都不可含有__重复元素__



#### HashSet

__本质__: HashMap

```java
public HashSet() {
        map = new HashMap<>();
}
```



特点：

1. HashSet可存放任意类型元素、但元素不可重复(==HashCode、equals()==)

2. 无序。

3. 本质: 使用HashMap  *Node[] table*  字段进行存储数据，每一个数据都是Node节点。

4. 组成: ==Node[] table 数组 + 单向链表+ 红黑树==

5. HashSet add()

   1. __添加元素位置由数组长度 + 元素keyhash值决定__
   2. 无法添加重复元素
   3. 一定程度后会进行树化

   ```java
   public boolean add(E e) {
   		//使用HashMap的put函数添加
           return map.put(e, PRESENT)==null;
   }
   
   //Map.put源码:
   public V put(K key, V value) {
       	//计算不同key的hash值确定索引
           return putVal(hash(key), key, value, false, true);
   }
   
   final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                      boolean evict) {
           Node<K,V>[] tab; Node<K,V> p; int n, i;
       	//判断是否为第一次添加元素
           if ((tab = table) == null || (n = tab.length) == 0)
               //进行扩容  
               n = (tab = resize()).length;
       	// 根据hash值计算在数组中的索引
           if ((p = tab[i = (n - 1) & hash]) == null)
               tab[i] = newNode(hash, key, value, null);
           else {
               Node<K,V> e; K k;
               //！！！   比较两者是否相同的根本根据 地址  equals方法。
               if (p.hash == hash &&
                   ((k = p.key) == key || (key != null && key.equals(k))))
                   e = p;
               else if (p instanceof TreeNode)
                   e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
               else {
                   //在链表中寻找位置
                   for (int binCount = 0; ; ++binCount) {
                       //找不到相同节点，则添加元素
                       if ((e = p.next) == null) {
                           p.next = newNode(hash, key, value, null);
                           //包括数组元素本身 + 链表上的元素 > 9
                           if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                               //红黑树化
                               treeifyBin(tab, hash);
                           break;
                       }
                       //找到了相同的Node节点
                       if (e.hash == hash &&
                           ((k = e.key) == key || (key != null && key.equals(k))))
                           break;
                       p = e;
                   }
               }
               if (e != null) { // existing mapping for key
                   V oldValue = e.value;
                   if (!onlyIfAbsent || oldValue == null)
                       //Value替换
                       e.value = value;
                   afterNodeAccess(e);
                   return oldValue;
               }
           }
           ++modCount;
           if (++size > threshold)
               resize();
           afterNodeInsertion(evict);
           return null;
       }
   final Node<K,V>[] resize() {
           Node<K,V>[] oldTab = table;
           int oldCap = (oldTab == null) ? 0 : oldTab.length;
           int oldThr = threshold;
           int newCap, newThr = 0;
       	//不是第一次扩容
           if (oldCap > 0) {
               //数组最大长度判断
               if (oldCap >= MAXIMUM_CAPACITY) {
                   threshold = Integer.MAX_VALUE;
                   return oldTab;
               }
               else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                        oldCap >= DEFAULT_INITIAL_CAPACITY)
                   //数组扩容原来得两倍
                   //threshold也成原来得两倍
                   newThr = oldThr << 1; // double threshold
           }
           else if (oldThr > 0) // initial capacity was placed in threshold
               newCap = oldThr;
           else {//第一次扩容            
   		   // zero initial threshold signifies using defaults
               //指定第一此扩容大小
               newCap = DEFAULT_INITIAL_CAPACITY;
               newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
           }
           if (newThr == 0) {
               float ft = (float)newCap * loadFactor;
               newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                         (int)ft : Integer.MAX_VALUE);
           }
           threshold = newThr;
           @SuppressWarnings({"rawtypes","unchecked"})
       		// 真正扩容的代码
               Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
           table = newTab;
           if (oldTab != null) {
               //扩容后的复制
               for (int j = 0; j < oldCap; ++j) {
                   Node<K,V> e;
                   if ((e = oldTab[j]) != null) {
                       oldTab[j] = null;
                       if (e.next == null) 
                           newTab[e.hash & (newCap - 1)] = e;
                       else if (e instanceof TreeNode)
                           ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                       else { // preserve order
                           Node<K,V> loHead = null, loTail = null;
                           Node<K,V> hiHead = null, hiTail = null;
                           Node<K,V> next;
                           do {
                               next = e.next;
                               if ((e.hash & oldCap) == 0) {
                                   if (loTail == null)
                                       loHead = e;
                                   else
                                       loTail.next = e;
                                   loTail = e;
                               }
                               else {
                                   if (hiTail == null)
                                       hiHead = e;
                                   else
                                       hiTail.next = e;
                                   hiTail = e;
                               }
                           } while ((e = next) != null);
                           if (loTail != null) {
                               loTail.next = null;
                               newTab[j] = loHead;
                           }
                           if (hiTail != null) {
                               hiTail.next = null;
                               newTab[j + oldCap] = hiHead;
                           }
                       }
                   }
               }
           }
           return newTab;
       }
   ```

6. ==扩容机制==: 

   1. 若添加元素数量超过 __threshold__ 则进行两倍扩容 newCap = oldCap << 1;newThe = oldTre << 1;
   2. ==jdk8==: 若某一条链表中(*包括数组节点本身*)长的 >= 9 时，若table长度小于64则进行扩容。否则，将该条链表进行红黑树化。

7. HashSet  remove()

   1. __HashMap移除元素是基于传入参数的hash决定得__
   2. 删除元素时要关注原始数据是否存在更改，传入参数数据是否已存在与table表中

   ```java
   public boolean remove(Object o) {
           return map.remove(o)==PRESENT;
   }
   
   //map.remove()
   public V remove(Object key) {
           Node<K,V> e;
           return (e = removeNode(hash(key), key, null, false, true)) == null ?
               null : e.value;
   }
   ```

   



#### LinkedHashSet（继承HashSet）

本质: *数组 + 双向链表*

特点：

	1. 计算hash值，计算数组中元素存在位置。
	1. 有head、tail字段指向头节点和尾节点
	1. Node节点有存放前驱节点、后继节点的引用。
	1. 扩容机制与HashSet一致。

![image-20221023103948586](C:\Users\Surpirse\AppData\Roaming\Typora\typora-user-images\image-20221023103948586.png)





#### TreeSet

本质：TreeMap

```java
public TreeSet() {
        this(new TreeMap<E,Object>());
}
```

特点:

1. 可根据一定的准则将输入数据进行排序(本质是树).
   ```java
   public TreeSet(Comparator<? super E> comparator) {
           this(new TreeMap<>(comparator));
   }
   public TreeMap(Comparator<? super K> comparator) {
           this.comparator = comparator;
   }
   ```

2. 可使用匿名内部类传入自定义comparator

3. TreeMap使用__put(K, key, V value)__函数时，若存在Key值相同(*Comparator*)的元素则进行替换。
   ```java
   public V put(K key, V value) {
           Entry<K,V> t = root;
           if (t == null) {
               //注意多态向上转型，若Key为一个对象则该类一定要实现Comparable接口
               compare(key, key); // type (and possibly null) check
               root = new Entry<>(key, value, null);
               size = 1;
               modCount++;
               return null;
           }
           int cmp;
           Entry<K,V> parent;
           // split comparator and comparable paths
           Comparator<? super K> cpr = comparator;
       	//使用自定义comparable
           if (cpr != null) {
               do {
                   parent = t;
                   cmp = cpr.compare(key, t.key);
                   if (cmp < 0)
                       t = t.left;
                   else if (cmp > 0)
                       t = t.right;
                   else
                       return t.setValue(value);
               } while (t != null);
           }
           else {
               //使用Key数据类型本身的compareTo()
               if (key == null)
                   throw new NullPointerException();
               @SuppressWarnings("unchecked")
               	//注意多态向上转型，若Key为一个对象则该类一定要实现Comparable接口
                   Comparable<? super K> k = (Comparable<? super K>) key;
               do {
                   parent = t;
                   cmp = k.compareTo(t.key);
                   if (cmp < 0)
                       t = t.left;
                   else if (cmp > 0)
                       t = t.right;
                   else
                       //若为相同Key则替换值
                       return t.setValue(value);
               } while (t != null);
           }
           Entry<K,V> e = new Entry<>(key, value, parent);
           if (cmp < 0)
               parent.left = e;
           else
               parent.right = e;
           fixAfterInsertion(e);
           size++;
           modCount++;
           return null;
       }
   ```

   

4. 仅允许存在一个key为null的数据

5. 在__TreeMap__中，传入的key对象必须实现*__Comparable__*接口。

6. 数据元素Key不允许为null













### Map(HashMap、LinkedHashMap、Hashtable、Properties、TreeMap)

#### HashMap

> 作用已在HashSet中说明了很清楚了，这里添加一些细节

1. HashMap __put(K, V)__  的使用，若K存在，则更新原table表中的值，若不存在则添加。

2. HashMap Key不能重复。可以使用null主键,null Value

3. entrySet()
   ```java
   transient Set<Map.Entry<K,V>> entrySet;
   ```

   返回一个Set集合，在HashMap中Node类实现了Entry接口。

   集合元素运行类型为Node。这里用到了接口的多态。

   4.keySet()返回所有的key

5. HashMap线程不安全







#### LinkedHashMap

> 与LinkedHashSet一致

存放数据为(Key, Value)



#### Hashtable

1. 存放的数据为键值对
2. 键和值都不能为null
3. Hashtable是线程安全的。







#### Propertites

1. 以键值对保存数据





#### TreeMap

> 作用与TreeSet一致

注意: Key不能为null而Value可以为null
