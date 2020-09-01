# HashMap

## 类介绍：

​	HashMap 是Java的一个集合类，继承了AbstractMap类，同时实现了Map、Cloneable、Serializable接口

```java
public class HashMap<K,V> extends AbstractMap<K,V>  implements Map<K,V>, Cloneable, Serializable 
```

## 静态不可变的变量

```java
//默认的初始容量 - 必须是二的幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//最大容量，如果一个较高的值是隐式地通过或者与参数的构造函数指定使用。 2的30次方
static final int MAXIMUM_CAPACITY = 1 << 30;

//在构造函数中未指定时使用的负载系数
static final float DEFAULT_LOAD_FACTOR = 0.75f;


//链表与红黑树互相转换的阈值  使用一棵树，而不是列表箱中的在库计数阈值。 与至少此多个节点将一个元素增加到仓当箱被转换为树。 该值必须大于2和应至少为8，在树去除大约收缩时转换回普通仓假设啮合
static final int TREEIFY_THRESHOLD = 8;

//为resize操作过程中untreeifying一个（分）仓的仓计数阈值。 应小于TREEIFY_THRESHOLD，和最多6与下除去收缩检测啮合
static final int UNTREEIFY_THRESHOLD = 6;

//最小的表容量，其二进制位可treeified。 （否则表如果在一个仓节点过多调整。）至少应为4 * TREEIFY_THRESHOLD来调整大小和treeification阈值之间避免冲突。
static final int MIN_TREEIFY_CAPACITY = 64;

```

## 构造函数

```java
//用指定的初始容量和负载因子的空HashMap
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //如果初始容量大于最大容量则使用最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
//用指定的初始容量和默认加载因子（0.75）的空HashMap
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
//构造具有默认初始容量（16）和默认负载因数（0.75）的空HashMap
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // 指定默认加载因子为0.75
}
//构造一个具有相同的映射关系与指定Map一个新的HashMap。 HashMap中与默认负载因数（0.75）和初始容量足以容纳在指定的地图的映射创建
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

```

## 静态内部类

```java
//树的节点类
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//节点的哈希值
    final K key;
    V value;
    Node<K,V> next;//下一个节点

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        //key的哈希值与value的哈希值进行异或运算得到节点的哈希值
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;//保存旧值
        value = newValue;//设置新值
        return oldValue;//返回旧值
    }

    public final boolean equals(Object o) {
        if (o == this)//两者地址相同，返回true
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

## HashMap::put(K key, V value) 方法

### put方法调用了putVal()方法，其中有个方法值得注意：<u>*hash(key)*</u>

```java
//参数 key/value
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
//计算hash
static final int hash(Object key) {
    int h;
    //key不为null,将key的哈希值右移16位忽略符号，然后与key的哈希值做异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

###  putVal方法

#### 参数列表

- hash：key的哈希键
- key：存储的key 键
- value：存储的值
- onlyIfAbsent：如果为true，不改变现有的值
- evict：如果为false，该表是在创建模式

```java
//hash 
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  boolean evict) {
    //定义了Node类型的数组tab和p
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //table 定义 transient Node<K,V>[] table;
    if ((tab = table) == null || (n = tab.length) == 0){
     	// 哈希表是空的话，构建，进行扩容
        n = (tab = resize()).length;
    }
    //
    //i = (n - 1) & hash  表中元素个数与hash进行 位与运算产生下标i,并且判断改下标处是否已有元素（是否hash碰撞）
    if ((p = tab[i = (n - 1) & hash]) == null){
        // 没有元素，直接在对应位置上构造一个新的节点即可
        tab[i] = newNode(hash, key, value, null);
    }
    else {
        Node<K,V> e; K k;
        if (p.hash == hash && //已有元素与新值hash是否相同
            ((k = p.key) == key || (key != null && key.equals(k))))//两者key是否相同，如果相同则是修改值
            e = p;
        else if (p instanceof TreeNode)
            //新增树节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //对链表p进行遍历，判断链表中存在是否存在该 key/value
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {//不存在该key/value
                    p.next = newNode(hash, key, value, null);//链表再次增加一个节点
                    if (binCount >= TREEIFY_THRESHOLD - 1)  // -1 for 1st
                        treeifyBin(tab, hash);//是否超出阈值，需要（数组+链表）转红黑数
                    break;
                }
                if (e.hash == hash && //链表中存在该key/value
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;//指针移动
            }
        }
        if (e != null) { // existing mapping for key
            //记录旧值
            V oldValue = e.value;
            //onlyIfAbsent=false 或者 旧值为null 
            if (!onlyIfAbsent || oldValue == null){
                e.value = value; //修改旧值
            }
            afterNodeAccess(e);//访问了节点e 
            return oldValue;//返回旧值
        }
    }
    ++modCount;//记录对HashMap的修改次数
    if (++size > threshold)//是否需要扩容
        resize();//扩容
    afterNodeInsertion(evict);//节点插入后的操做，当前Map的状态,evict=false该表是在创建模式。
    return null;
}
```

通过逐行代码解析，我们大概了解了HashMap::put() 的执行流程如下图所示![HashMap-put()流程图](F:\我的\java\HashMap-put()流程图.png)

