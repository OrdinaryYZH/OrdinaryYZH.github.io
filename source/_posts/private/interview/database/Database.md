1. 索引有什么用？如何建索引？
   索引的目的在于提高查询效率
2. 数据库优化相关
   不太懂
3. 数据库事务隔离级别？
   * 读未提交（Read uncommitted），问题：脏读
   * 读提交（read committed），问题：不可重复读
   * 可重复读（repeatable read），问题：幻读
   * 串行化（Serializable）
4. ​



### 索引

#### 组合索引的最左优先原则

MySQL组合索引（复合索引）的最左优先原则。最左优先就是说组合索引的第一个字段必须出现在查询组句中，这个索引才会被用到。只要组合索引最左边第一个字段出现在Where中，那么不管后面的字段出现与否或者出现顺序如何，MySQL引擎都会自动调用索引来优化查询效率。 

根据最左匹配原则可以知道B-Tree建立索引的过程，比如假设有一个3列索引(col1,col2,col3),那么MySQL只会会建立三个索引(col1),(col1,col2),(col1,col2,col3)。 

所以题目会创建三个索引（plat_order_id）、（plat_order_id与plat_game_id的组合索引）、（plat_order_id、plat_game_id与plat_id的组合索引）。根据最左匹配原则，where语句必须要有plat_order_id才能调用索引（如果没有plat_order_id字段那么一个索引也调用不到），如果同时出现plat_order_id与plat_game_id则会调用两者的组合索引，如果同时出现三者则调用三者的组合索引。 

> 考察组合索引的知识。有一个最左优先的原则，组合索引(a, b, c)，会建立三个索引, (a), (a, b), (a, b, c)。 
>
>   在查询语句中， 
>
>   1)where a = xxx  
>
>   2)或者 where a = xxx and b = yyy 
>
>   3)或者where a = xxx and b = yyy and c = zzz 
>
>   4)或者 where b = yyy and c = zzz and a = xxx 
>
>   都可以使用索引。第4）中情况可以优化成第3）种。 
>
>   不包含a的情况，则用不到索引。例如where b = yyy and c = zzz