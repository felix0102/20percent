# MySQL 索引与优化必知必会

### MySQL 索引基本概念

#### 1.1 什么是索引？

在关系数据库中，索引是一种单独的、物理的对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。（百度百科）

从上面的定义中我们可以知道，索引是一种存储结构；是数据库表中一列或若干列值的集合，或者是指向表中数据页物理标识的逻辑指针清单。在日常生活中索引就像是一本书的目录，一个商品的使用说明书，我们可以跳过大部分不关心的内容，直奔我们需要知道的主题。目录和使用说明书可以节省我们很多时间，索引也同样用来提高数据库的检索速度，提高数据库性能。

例如：有一张 student 表，其中有 5W 条记录，记录这 5W 个学生的信息。其中有一个 sno 的字段记录每个学生的学号，现在想要查询出学号为 XXXX 的人的信息。

在没有索引的情况下，我们需要一条一条的遍历表中的数据。在查询到匹配的数据后，先将该数据放入到结果集中，再继续查询直到将全部的数据查询完毕。

而在有索引的情况下， 可以将 sno 的 key 的值放在一个 n 叉树上（ BTree ）。查询数据时，会在 B 树上根据 sno 的值进行条件查询， 而不需要全局遍历。

#### 1.2 索引的优点

1. 索引的最明显的优点就是减少查询的速度，提高数据库的效率。
   - 通过索引查询可以缩短数据检索的时间
   - 通过索引可以加快表与表之间的关联查询时间
2. 为排序或者分组的字段增加索引，可以提升排序和分组的效率。

#### 1.3 索引的缺点

1. 创建索引的时间成本会随着数据量的增大而增大。
2. 索引创建之后，在对数据库表中的数据进行增、删、改等操作之后，相应的索引也需要进行维护，降低了数据维护的速度。
3. 从本质上来讲，索引是通过空间来换取时间。也就是说，在我们缩短查询的时间成本的同时，势必要牺牲磁盘空间成本来存储索引。

#### 1.4 索引的应用

根据上面所讲的优缺点我们知道，索引是需要在一定的场景下使用才能真正的提高效率的，需要根据具体情况进行分析决定是否使用索引。

1. 表数据量较大且响应时间不能满足需求的时候，可以考虑使用索引。
2. 索引建议创建在变动频率较小，且值内容较多的字段上。

### 索引分类

#### 2.1 物理角度

1. 集聚索引：决定数据在磁盘上的物理排序，一个表只能有一个聚集索引。
2. 非集聚索引：不决定数据在磁盘的物理排序，索引上只包含建立索引的数据和一个类似聚集索引物理排序指针一样的定位符，通过这个定位符可以找到数据。

#### 2.2 使用角度

在本小节，我们重点从应用开发中我们经常设置的索引类型来进行介绍。 总结来说比较常用的索引类型：普通索引、复合索引、唯一索引、主键索引；不常用的一些索引类型有：全文索引、空间索引。

##### **2.2.1 普通索引**

最基本的索引，它没有任何限制。普通索引是使用最为频繁的索引。在使用场景中我们选择建立普通索引要关注下面几点：

- 目前索引的总数，索引总数太多会对数据新增造成效率影响。
- 建立索引的字段的查询频次，以及表的总行数大小。

##### **2.2.2 实战总结**

**场景 1 **

在销售人员管理系统中，销售人员信息表存储了销售员工的基本信息，包括：用户 ID ，名称，部门，等级，职位，区域，电话号码，邮箱等。

在进行工单派发与查询的时候一个经常需要使用的查询场景，根据销售电话号来查询该销售其他的详细基本信息。 随着销售人员的不断增加，根据电话号码字段来查询这个语句性能不断下降，更加可怕的是随着业务不断发展，这种查询需求不断增加。 因此我们需要针对销售的电话号码列建立普通索引。

普通索引的建立能够有效地提高热频次查询语句的效率。避免慢 SQL 的产生。

在这里大家注意一下，慢 SQL 的治理，很大程度上需要借助索引来进行辅助。

##### **2.2.3 复合索引**

**概念简介：**在多个字段上创建索引，可以加速复合条件查询。复合索引也叫做联合索引。 复合索引的创建，我们需要寻找多条件频繁查询的语句来进行设置。

##### **2.2.4 实战总结**

**场景 1 **

交易中心的订单表，存储了用户交易订单相关的信息。表中重要的字段信息有用户 ID 、订单 ID 、商品 ID 、商品个数、支付金额、订单状态、交易日期、创建日期、付款时间、退款时间、物流 ID 等。

用户会有非常多的场景来查询某个订单状态下的订单列表。 比如我们经常会查询已支付但未到货的订单信息。 这就涉及到三个字段：用户 ID 、订单状态。

这时候我们就可以针对（用户 ID ，订单状态）这两个字段进行复合索引的创建。

当然复合索引随着业务不同，会有各种不同的配置。 使用复合索引的时候需要注意： 1.复合索引的最左前缀特点。复合索引由字段 (a,b,c) ，那么复合索引的查询可以支持 a | a,b | a,b,c 3 种组合进行查找，但不支持 b,c 进行查找。

##### **2.2.5 唯一索引**

一种唯一性索引。索引列值要唯一，但允许有空值。如果是组合索引，则组合后的列值必须唯一。

##### **2.2.6 实战总结**

**场景 1**

用户支付信息表，有一列为 pay_num 。这一类表示用户每次支付的唯一 ID ，是所有用户支付凭证记录。因此 pay_num 列设置唯一索引，来保证避免 pay_num 的重复。

与此同时，在业务中不同业务线也会经常根据 pay_num 来查询详细的支付详情信息。因此索引的建立有助于检索效率的提升。

在这个场景下，唯一索引起到了两个作用：避免重复保证唯一性，提高检索效率。

##### **2.2.7 主键索引**

**概念简介：**一种特殊的唯一索引。用于唯一标识数据表中某条记录，不允许有空值，一般用 primary key 来控制。

**实战总结：**

这个实战场景就不给大家举例子了。只提供一些创建准则：每个业务表中尽量都去创建一个主键索引，并且尽量使用自增 ID 来表示主键索引。

主键索引注意点：

1. 可以优化数据插入性能，避免 page 分裂，提升空间和内存使用率
2. 较短数据类型适合设置主键， Innodb 引擎普通索引会保存主键的值，因此较短的数据类型可以减少磁盘空间使用

##### **2.2.8 全文索引**

**概念简介：**全文索引是一种特殊类型的索引，它查找的是文本中的关键词，而不是直接比较索引中的值。全文索引更像是一个搜索引擎而不是简单的 where 语句的参数匹配。

因为在业务中不建议使用全文索引，所以就不进行相关的实战介绍。

##### **2.2.9 空间索引**

**概念简介：**空间索引是对空间数据类型的字段建立的索引，MySQL 中的空间数据类型有 4 种，分别是 geometry、point、linestring、polygon。空间索引只能在存储引擎为 MyISAM 的表中创建，创建空间索引的列必须声明不允许为空。

### MySQL 索引实现方式

#### 3.1创建索引

##### **3.1.1 创建表同时创建索引**

```
CREATE TABLE table_name [col_name data type]
    [unique:fulltext:spatial]
    [index:key]
    [index_name]
    (col_name[length])
    [asc:desc]
```

**[unique : fulltext : spatial]** ：指定索引类型，唯一索引：全文索引：空间索引。 **[index : key]** ：index 和 key 都可以用来指定索引，区别是 index 可以给索引命名， key 则默认字段名作为索引名称。 **[index\*name]\*** *：索引的名称，若不指定默认为 col*name 的值，由前一个参数是 index 还是 key 决定。 **(col\*name[length])\*** *：需要创建索引的字段，不需是表中定义的列， length 为可选参数，只有在 col*name 是字符串串类型的时候才能指定长度。

- 创建普通索引

```
CREATE TABLE order id INT NOT NULL AUTO_INCREMENT, 
    order_code VARCHAR(255) NOT NULL,
    price BIGDECIMAL NOT NULL,
    state INT NOT NULL, 
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    pro_code VARCHAR(255) NOT NULL,
    extension VARCHAR(255),
    INDEX (order_code));
```

- 创建复合索引

```
CREATE TABLE order (id INT NOT NULL AUTO_INCREMENT, 
    order_code VARCHAR(255) NOT NULL,
    price BIGDECIMAL NOT NULL,
    state INT NOT NULL, 
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    pro_code VARCHAR(255) NOT NULL,
    extension VARCHAR(255),
    INDEX (id, order_code, state);
```

- 创建唯一索引

```
CREATE TABLE order (id INT NOT NULL AUTO_INCREMENT, 
    order_code VARCHAR(255) NOT NULL,
    price BIGDECIMAL NOT NULL,
    state INT NOT NULL, 
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    pro_code VARCHAR(255) NOT NULL,
    extension VARCHAR(255),
    UNIQUE INDEX (order_code));
```

- 创建主键索引

```
CREATE TABLE order (id INT NOT NULL AUTO_INCREMENT, 
    order_code VARCHAR(255) NOT NULL,
    price BIGDECIMAL NOT NULL,
    state INT NOT NULL, 
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    pro_code VARCHAR(255) NOT NULL,
    extension VARCHAR(255),
    PRIMARY KEY(id));
```

- 创建全文索引

```
CREATE TABLE order (id INT NOT NULL AUTO_INCREMENT, 
    order_code VARCHAR(255) NOT NULL,
    price BIGDECIMAL NOT NULL,
    state INT NOT NULL, 
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    pro_code VARCHAR(255) NOT NULL,
    extension VARCHAR(255),
    FULLTEXT INDEX (order_code))
    ENGINE=MyISAM;
```

###### **3.1.2 在已有表上创建索引**

```
ALTER TABLE table_name ADD
    [unique:fulltext:spatial]
    [index:key]
    [index_name]
    (col_name[length])
    [asc:desc]
```

- 修改普通索引

```
ALTER TABLE order ADD INDEX (order_code(255));
CREATE INDEX order_code on order(order_code);
```

- 创建复合索引

```
ALTER TABLE order ADD INDEX (id, order_code(255), state);
CREATE INDEX order_code on order(id, order_code， state);

```

- 创建唯一索引

```
ALTER TABLE order ADD UNIQUE INDEX (order_code);
CREATE UNIQUE order_code on order(order_code);
```

- 创建主键索引

```
ALTER TABLE order ADD PRIMARY KEY(id));
```

- 创建全文索引

```
ALTER TABLE order ADD FULLTEXT INDEX (order_code(255));

```

#### 3.2 查询和删除索

##### **3.2.1 查询索引**

```
SHOW INDEX FROM table_name;
SHOW INDEX FROM order;
```

##### **3.2.2 删除索引**

```
DROP INDEX index_name on table_name;
DROP INDEX order_code ON order;
ALTER TABLE table_name DROP INDEX index_name;
ALTER TABLE order DROP PRIMARY KEY;
ALTER TABLE order DROP INDEX order_code;
```

### MySQL 索引的问题及优化方案

1. 前导模糊查询利用不到索引。

   例如： select * from order where extension like '%XXX'; 该 SQL 在查询索引字段的时候，由于查询条件开始是模糊的，会导致索引失效，会导致查询全局扫描或者全索引扫描。因此在页面严禁做模糊或者全模糊搜索，如果需要可以通过使用搜索引擎来解决。

2. union、in、or 都能够命中索引，建议使用 in。

   ```
   select * from order where id = 1 
   union all
   select * from doc where id = 2;
   ```

   使用 union 可以命中索引，消耗 CPU 也是最少的，但是一般不这么写 SQL。

   ```
   select * from order where id in (1, 2);
   ```

   in 同样可以命中索引，查询时消耗的 CPU 比 union all 要多一些，但是通常情况下可以忽略不计，建议使用这种方式

   ```
   select * from order where id = 1 or id = 2;
   ```

   在新版的 MySQL 中 or 可以命中索引，但是查询时消耗 CPU 比 in 还要多，因此不建议频繁使用 or。

3. 负向条件查询不能使用索引，可以优化为 in 查询。

   负向条件包含 ：<>（!=）、not in、 not like、 not exists等 例如：

   ```
   select * from order where id not in (1, 2)；
   ```

   可以优化为 in 查询：

   ```
   select * from order where id in （3, 4, 5);
   ```

4. 创建索引时避免以下错误观念

   第一、索引多多益善，过多的索引会占用更多的系统空间，而且维护起来难度也会相应增高； 第二、索引宁缺毋滥，认为索引会消耗空间，降低新增和更新的速度； 第三、抵制唯一索引，在应用层面通过“先查后插”来对唯一性进行控制； 第四、优化索引的时间不正确，过早优化索引，可能会因为不了解系统而优化不完全；过晚的优化索引又可能增加修改索引的工作量。

   索引多多益善和索引宁缺毋滥是使用索引的两种极端表现，还是需要根据具体的情况分析如何使用索引，才能提高数据库的使用效率。

5. 超过三个表最好不要 join。

   - join 字段的数据类型，要求必须一致；
   - 多表关联查询时，被关联的字段必须有索引。

6. 如果明确知道只有一条结果返回，limit 1 能够提高效率。

   在知道只有一条结果的时候，我们需要明确的告诉数据库我们只要查这一条结果，让数据库停止继续查询。

   ```
   select * from order where user_id = ‘XXX’ limit 1；
   ```

7. 业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引。

   根据墨菲定律，如果在数据库中没有创建唯一索引，即使在应用层做了非常完善的数据校验和控制，也会有脏数据产生；而且与明显提高查询的速度来比较，在给有唯一索引的数据库添加数据时降低的速度几乎可以忽略不计。

8. 联合索引最左前缀原则（又叫最左侧查询）

   如果在(a,b,c)三个字段上建立联合索引，那么它能够在 a | (a,b) | (a,b,c) 三个地方生效。 例如：

   ```
   select * from order where order_code = 'XXXX' and state = XX;
   ```

   可以建立 (order*code, state) 的联合索引。 在业务上根据订单的编号作为查询条件的需求有很多，但是几乎没有仅用订单状态来查询的需求，因此是创建 (order_code, state) 的联合索引，而不是 (state, order*code) 的联合索引。

   建联合索引的时候，区分度最高的字段在最左边。 存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如 where a > XX and b = XX的时候，那么即使 a 的区分度更高，也必须把 b 放在索引的最前列。 最左侧查询，并不是指 SQL 语句的 where 查询条件顺序要和联合索引顺讯一致。 下面的 SQL 语句也可以命中 (order_code， state) 这个联合索引。

   ```
   select * from order where state = XX and order_code = 'XXXX';
   ```

   但我们还是建议 where 后查询条件的顺序与联合索引的顺序保持一致。 如果建立了 (a,b) 联合索引，就不必再单独建立 a 索引，如果建立了 (a,b,c) 联合索引，就不必再单独建立 a、(a,b) 索引。

9. 范围列可以用到索引（联合索引必须是最左前缀）。

   范围列可以命中索引；在联合索引中，如果存在两个或两个以个上范围列，则最多只有一个可以命中索引（命中原则为：最左前缀）。 范围条件包括：>、>=、<=、<、between 等。 例如有 (id，order*code，end*date) ，那么下面的 SQL 中 id 可以命中索引，而 price 和 from_date 则使用不到索引。

   ```
   select  * from order where id < 10300 and order_code = 'XXXX' and end_date between '2019-01-01' and '2019-12-31';
   ```

10. 把计算放到业务层而不是数据库层。

    ```
    select * from order where price = (amount * unit_price);
    ```

    在上面的 SQL中，即使 price 上创建了索引，也会全表扫描。这是因为在字段上计算是不能够命中索引的。 可以把计算放到业务层，这样做既节省数据库的对 CPU 的占用，还可以优化查询缓存。

11. 强制类型转换会全表扫描

    如果 varchar 类型在查询的时候不加引号，该值会被强制转成 int 类型，而强转之后的字段不能命中索引。 例如：

    ```
        select * from order_code = XXXX；   //不能命中索引
        select * from order_code = 'XXXX';    //可以命中索引
    ```

12. 更新十分频繁、数据区分度不高的字段上不宜建立索引。

    MySQL目前主要有以下几种索引方法： B-Tree，Hash，R-Tree。不论用那种索引方法，更新频繁都会大大降低数据库的性能。

    一般字段区分度在 80% 以上的时候就可以在该字段上建立索引了，区分度可以使用 count(distinct( 字段名称 )/ count(*)) 来计算。像是“性别”这种区分度很低的字段，建立索引的意义也是非常小的。

13. MySQL 主要提供 2 种方式的索引： B-Tree 索引， Hash 索引

    B 树索引具有范围查找和前缀查找的能力，对于有 N 节点的 B 树，检索一条记录的复杂度为 O(LogN)。与二分查找相同。

    哈希索引只能做等于查找，但是无论多大的 Hash 表，查找复杂度都是 O(1)。

    显然，如果值的差异性大，并且以等值查找 (=、 <、>、in) 为主，Hash 索引是更高效的选择，它有 O(1) 的查找复杂度。

    如果值的差异性相对较差，并且以范围查找为主，B 树是更好的选择，它支持范围查找。

14. 建立索引的列，不允许为 null。

    a.单列索引无法储 null 值，复合索引无法储全为 null 的值。需要使用 not null 约束以及默认值。 b.查询时，采用 is null 条件时，不能命中索引，只能全表扫描。

    索引无法存储 null 值的原因如下： a. 索引是有序的。null 值进入索引时，无法确定其应该放在哪里。 (null 值不能进行比较，无法确定 null 出现在索引树的叶子位置的节点位置。) b. 如果需要把空值存入索引，方法有二：其一，把 null 值转为一个特定的值，在 where 中检索时，用该特定值查找。其二，建立一个复合索引。

15. 单索引字段数不允许超过 5 个。

    当单索引的字段数超过 5 个的时候，就已经没有过滤数据的效果了。

16. 使用短索引（又叫前缀索引）来优化索引。

    前缀索引 对文本的前一部分建立索引，这样可以节约空间，也可以优化查询效率。可以使用 count(distinct left ( 列名, 索引长度 )) / count(*) 来计算前缀索引的区分度。 其缺点是不能使用 ORDER BY 和 GROUP BY 操作，也不能用于覆盖索引，全字段建立索引有时候没必要，就可以根据实际文本区分度决定索引长度即可。 例如：

    ```
    select * from order where order_code = 'XXXX' and extension = '110108XXXXXXXXXXXX'；
    ```

    就可以创建索引：(order_code， extension(6))

17. SQL 性能优化 explain 中的 type：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好。

    all：表示全表扫描 index ：扫描顺序是按照索引的顺序的全表全扫，速度非常慢。 consts ：单表中最多只有一个匹配行（主键或者唯一索引），在优化阶段即可读取到数据。 ref ：查找条件列使用了索引而且不为主键和 unique。使用普通的索引 (Normal Index) range ：对索引进行范围检索。

### MySQL 索引与锁的实现

#### 5.1 锁用来干嘛的？

当多个线程对数据库进行并发读写操作的时候，可能会带来数据不一致的问题。锁主要就是在并发环境下保证数据库的一致性。

> 从应用角度来说，锁分为：悲观锁，乐观锁两种 从锁控制范围来说，锁分为：表级锁，行级索 从互斥性角度来说，锁分为：共享锁，排他锁

从这些名词上都可以看出来，MySQL 的锁机制是一个比较难理解的知识点。

#### 5.2 乐观锁

乐观锁的特点是先进行具体的业务功能实操作，等到业务功能执行完毕， 需要实际更新数据的时候再最后一步拿一下锁。

乐观锁实现完全是应用层面自己的事情，不需要数据库提供特殊的支持。 比较常见的做法，就是在需要锁的数据上增加一个版本号，然后当运行完业务功能之后， 提交结果的时候对比版本号是否发生变化。

```
update set stock=stock-1,version = new_version where varsion=old_version

```

#### 5.3 悲观锁

悲观锁特点是必须先获取锁，再进行业务操作，一般来说在数据库上的悲观锁都是数据库本身提供的能力。 例如我们用： select….for update 这个语句操作就可以实现悲观锁。 当数据库执行 select for update 时会获取被 select 中的数据行的行锁，因此其他并发执行的 select for update 如果试图选中同一行则会发生排斥（需要等待行锁被释放），因此达到锁的效果。

#### 5.4 表/行级锁

**表锁 ** 直接锁定整张表，锁定期间，其他线程无法对该表进行写操作。锁维护代价比较小，锁的范围比较大，所以锁的冲突概率比较高，并发程度比较差。

**行锁 ** 对指定的记录行加锁，其它线程无法对该行记录同时进行写操作，但可以对表中其他行记录做任何操作。 行级锁实现代价比较大，加锁效率比较低， 但锁定粒度相对比较小，所以不容易发生锁冲突现象，整体的并发度高。

锁的粒度随着存储引擎的类型不同有所区别：

InnoDB 同时支持行级锁和表级锁两个类型，并且通过索引列来查询的数据才使用行级索，否则是使用表级锁 MyISAM 只支持表级锁，不支持行级索

#### 5.5 共享/排他锁

**共享锁（ S 锁）：**也叫做读锁，读锁是共享的，多个事务可以同时获取同一个锁，但不允许其他客户修改。

**排他锁（ X 锁）：**从名字上可以看出不能与其他锁同时存在的锁。如果一个事务得到了某个数据行的排他锁，任何其他事务就不能再获取，必须等待。 也叫做写锁：写锁是排他的，写锁会阻塞其他的写锁和读锁。

### MySQL 索引常见面试题

1. MySQL 的存储引擎 MyISAM 和 InnoDB 的区别。

   - MyISAM 适合用于频繁查询的应用。表级锁，适合小数据，没有死锁，不支持事务，不支持外键
   - InnoDB 支持事务，适合插入和更新频繁的场景，支持行锁，当然也支持表锁，适合数据量比较大，并发量比较大的场景，支持外键

2. 数据库优化从几个方面思考？

   - SQL 语句的优化
   - 表结构优化，索引优化
   - 分库分表操作
   - 使用缓存，减少对于数据库的压力

3. 经常使用的索引类型有哪些？

   普通索引， 联合索引，唯一索引， 主键索引

4. 什么情况下，索引无法被使用？

   - 以 “%” 开头的 LIKE 语句，模糊匹配
   - OR 语句前后没有同时使用索引
   - 数据类型出现隐式转化（如 varchar 不加单引号的话可能会自动转换为 int 型）

5. 建立索引的原则有哪些？

   在最频繁使用的、用以缩小查询范围的字段上建立索引。 在频繁使用的、需要排序的字段上建立索引

6. 什么是事务？详细描述其特点

   事务：是一系列的数据库操作，是数据库应用的基本逻辑单位。 事务特性：

   - 原子性：即不可分割性，事务要么全部被执行，要么就全部不被执行。
   - 一致性或可串性。事务的执行使得数据库从一种正确状态转换成另一种正确状态。
   - 隔离性。在事务正确提交之前，不允许把该事务对数据的任何改变提供给任何其他事务。
   - 持久性。事务正确提交后，其结果将永久保存在数据库中，即使在事务提交后有了其他故障，事务的处理结果也会得到保存。

7. char 和 varchar 的区别是什么？

   - char(M) 类型的数据列里，每个值都占用 M 个字节，如果某个长度小于 M ， MySQL 就会在它的右边用空格字符补足．
   - varchar(M) 类型的数据列里，每个值只占用刚好够用的字节再加上一个用来记录其长度的字节（即总长度为 L+1 字节）

### 小结

本文向大家介绍了关于 MySQL 相关的基础知识。 希望对大家以后的工作有所帮助。