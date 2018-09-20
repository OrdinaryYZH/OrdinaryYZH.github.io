## 什么是Map

![](https://ws1.sinaimg.cn/large/8747d788gy1fve71o7mxgj20ts0k3k0i.jpg)



## Map常用方法

1. 查询
  ```java
  int size();
  boolean isEmpty();
  boolean containsKey(Object key);
  boolean containsValue(Object value);
  V get(Object key);
  ```

2. 修改
  ```java
  V put(K key, V value);
  V remove(Object key);
  ```

3. 批量操作
  ```java
  void putAll(Map<? extends K, ? extends V> m);
  void clear();
  ```

4. Views

  ```java
  Set<K> keySet();
  Collection<V> values();
  Set<Map.Entry<K, V>> entrySet();
  ```

5. Comparison and hashing

  ```java
  boolean equals(Object o);
  int hashCode();
  ```

6. Defaultable methods
  ...