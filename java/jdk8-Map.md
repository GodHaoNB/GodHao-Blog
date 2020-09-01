# Map解析

## 一、Map<K,V>接口方法、参数以及返回类型

```java
int size();	//获取元素个数
boolean isEmpty();//集合是否为空 是:true 否:false
boolean containsKey(Object key);//集合中是否存在键key 是:true 否:false
boolean containsValue(Object value);//集合中是否存在值value 是:true 否:false
V get(Object key); //获取键key对应的值value
V put(K key, V value);//向集合中插入键值对key-value,若是覆盖操作插入后返会oldValue，否则Null
V remove(Object key);//删除一个元素
void putAll(Map<? extends K, ? extends V> m);//集合合并，将一个集合的数据插入另一个集合
void clear();//清空所有元素
Set<K> keySet();//将集合中的key转换成Set集合并返回
Collection<V> values();//将集合中的value转换成Collection集合并返回
Set<Map.Entry<K, V>> entrySet();//包含了Map所有元素的实体的Set集合
boolean equals(Object o);
int hashCode();

//获取key的值，并传入一个默认值，如果Map中存在该值或者key存在则返回v,否则返回传入的默认值
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}
/**
*  action: @FunctionalInterface public interface BiConsumer<T, U>  
*  迭代器 函数式编程 
*/
default void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
        action.accept(k, v);//将K,V 传入action,没有返回值
    }
}
/**
* 所有值替换 函数式接口
*/
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    Objects.requireNonNull(function);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }

        // ise thrown from function is not a cme.
        v = function.apply(k, v);//获取v

        try {
            entry.setValue(v);
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
    }
}
//插入一条数据
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {//如果v 值不为null 则插入
        v = put(key, value);
    }

    return v;//返回v
}
//删除一个值 返回true或false
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    //如果 value与curValue的值不同或者（curValue的值为空并且key不存在）则删除失败返回false
    if (!Objects.equals(curValue, value) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    remove(key);
    return true;
}
//单个值替换 带条件的
default boolean replace(K key, V oldValue, V newValue) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, oldValue) ||//传入的旧值与map中的值不同或者map中的值为空并且key不存咋则替换失败返回false
        (curValue == null && !containsKey(key))) {
        return false;
    }
    put(key, newValue);//插入新值
    return true;
}
//不带条件的值替换
default V replace(K key, V value) {
    V curValue;
    if (((curValue = get(key)) != null) || containsKey(key)) {//值不为null或者key存在则替换成功
        curValue = put(key, value);
    }
    return curValue;
}
//插入key/value，函数式编程
default V computeIfAbsent(K key,
                          Function<? super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {// 如果集合中不存在该数据则插入
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) { 
            put(key, newValue);
            return newValue;
        }
    }

    return v;
}

default V computeIfPresent(K key,
                           BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {//如果 oldValue 不为空
        V newValue = remappingFunction.apply(key, oldValue);//传入两个参数，得到一个值
        if (newValue != null) {//若newValue不为空，则插入，否则移除
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}

default V compute(K key,
                  BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue = get(key);
	
    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue == null) {//如果 新值为null
        // delete mapping
        if (oldValue != null || containsKey(key)) { //
            // something to remove
            remove(key);
            return null;
        } else {
            // nothing to do. Leave things as they were.
            return null;
        }
    } else {
        // add or replace old mapping
        put(key, newValue);//插入
        return newValue;
    }
}
default V merge(K key, V value,
                BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
    remappingFunction.apply(oldValue, value);//通过运算（oldValue, value）得到一个新值
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```



## 二、Map.Entry<K,V> 接口方法

```java
interface Entry<K,V> {
    K getKey();
    V getValue();
    V setValue(V value);
    boolean equals(Object o);
    int hashCode();

    //返回一个Map.Entry以自然顺序对键进行比较的比较器。 可序列化的
    //K :Comparable<? super K>
    //V : Map 的V类型
    public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
        return (Comparator<Map.Entry<K, V>> & Serializable)// '&' jdk8的新语法：标识必须同时满足两个接口
            (c1, c2) -> c1.getKey().compareTo(c2.getKey());
    }

    //返回一个Map.Entry以自然顺序比较值的比较器。
    public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getValue().compareTo(c2.getValue());
    }

    //返回一个比较器，该比较器Map.Entry使用给定的键进行 比较Comparator。
    public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
        Objects.requireNonNull(cmp);//如果cmp 为null 抛出NullPointerException
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
    }

    //返回一个比较器，该比较器Map.Entry使用给定的值进行 比较Comparator。
    public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
        Objects.requireNonNull(cmp);
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
    }
}
```

