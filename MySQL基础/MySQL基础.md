[toc]

# MySQL数据类型

1. 数值类型
   1. 整数

      | 类型     | **大小** |
      | -------- | -------- |
      | tinyInt  | 1字节    |
      | smallInt | 2字节    |
      | medium   | 3字节    |
      | int      | 4字节    |
      | bigInt   | 8字节    |

   2. 浮点

      | 类型   | 大小  |
      | ------ | ----- |
      | float  | 4字节 |
      | double | 8字节 |

2. 字符串
   
   1. char
   2. varchar
   3. tinytext
   4. text
   
3. 日期
   1. date				yyyy-MM-dd
   2. time                HH:mm:ss
   3. datetime        yyyy-MM-dd HH:mm:ss
   4. timestamp

4. null

# 存储结构

- 数据库
  - 表空间	tablespace
    - 段	segment
      - 区	extent
        - 页	page 默认16kb
          - 行	row

`InnoDB中，1个区分配64个页`

`数据库I/O操作最小单位为页`

# 执行过程

## Oracle

1. 语法检查
2. 语义检查
3. 权限检查
4. 共享池检查
   1. 库缓存：决定是硬解析还是软解析
   2. 数据字典缓冲区
5. 优化器：硬解析
   1. 创建解析树
   2. 生成执行计划
6. 执行器

## MySQL

1. 查询缓存：8.0版本后舍弃
2. 解析器
3. 优化器
4. 执行器

# 存储引擎 :star2:

MyISAM：节约空间，速度快

InnoDB：安全性高，适合多用户操作

|          | MyISAM | InnoDB |
| -------- | ------ | ------ |
| 事务     | 不支持 | 支持   |
| 外键约束 | 不支持 | 支持   |
| 锁       | 表锁   | 行锁   |
| 全文索引 | 支持   | 不支持 |
| 表空间   | 较小   | 较大   |

# 表约束

1. 主键约束	非空唯一
2. 外键约束
3. 唯一性约束
4. 非空约束
5. default
6. check约束

# SELECT顺序

## 语法顺序

```sql
SELECT...FROM...
JOIN...ON...
WHERE...
GROUP BY... HAVING...
ORDER BY...
LIMIT...
```

## 执行顺序

```sql
CROSS JOIN... ON...
WHERE...
GROUP BY...
HAVING...
SELECT...
DISTINCT...
ORDER BY...
LIMIT...
```

# 事务

## 事务隔离级别 :star2:

> 脏读：
>
> 不可重复读
>
> 幻读：

|          | 脏读 | 不可重复读 | 幻读 |
| -------- | ---- | ---------- | ---- |
| 读未提交 | 允许 | 允许       | 允许 |
| 读已提交 | 禁止 | 允许       | 允许 |
| 可重复读 | 禁止 | 禁止       | 允许 |
| 串行     | 禁止 | 禁止       | 禁止 |

MySQL默认可重复读，Oracle、sqlServer默认读已提交

# 索引

加快查询效率的数据结构

## 索引种类

1. 普通索引
2. 唯一索引
3. 主键索引
4. 全文索引

一张表，可以有多个唯一索引；但是只能有一个主键索引

## 索引数据结构

MySQL的索引使用B+树结构

1. 索引列存储在叶子节点
2. 由于B+树特性，使得叶子节点的索引数据已经进行排序
3. 如果使用的是联合索引，当查询的字段与联合索引的列相匹配，则此次查询无需回表，称为`索引覆盖，此时查询效率较高。
4. 主键建议使用`自增int`类型。由于索引使用B+树结构，使得叶子节点为排好序的状态。增加数据时，如果主键为int类型，则数据直接插插在末尾叶子节点后即可，此时对整颗树影响最小。

## 创建索引条件

1. 字段数值具有唯一性
2. where条件字段
3. group by/order by字段
4. distinct字段

> 多个单列索引再条件查询时只会生效1个，默认最严格
>
> 多条键联合查询最好使用联合索引

## 索引失效

1. 对索引列进行表达式计算
2. 对索引列使用函数
3. where字句中，or前条件使用索引，or后条件未使用索引
4. 模糊查询
5. 不符合最左原则

## 索引缺点 :star2:

1. 占用磁盘空间
2. 降低写性能
3. 有多个索引时，索引选择需要浪费时间

# 锁

**保持数据一致性**

根据不同角度，锁有多种分类

1. 根据锁粒度
2. 根据锁管理划分
3. 根据锁思想划分

## 根据粒度划分

|      | 锁粒度 | 并发度 | 开销 | 枷锁速度 |
| ---- | ------ | ------ | ---- | -------- |
| 行锁 | 小     | 高     | 大   | 慢       |
| 页锁 | 中     | 中     | 中   | 中       |
| 表锁 | 大     | 低     | 小   | 快       |

锁数量有限，当某层级的锁数量超出阈值，就会触发锁升级，`自动升级到大粒度锁`

## 按锁管理划分

- 共享锁：S锁，其他用户可读，不可写。SELECT时使用共享锁
- 排他锁：独占锁。其他事务不可读写

## 按照思想划分

- 乐观锁：适合读操作比较多的场景
  - 版本号法
  - 时间戳法
- 悲观锁
  - 遵从上述粒度/思想的划分方法