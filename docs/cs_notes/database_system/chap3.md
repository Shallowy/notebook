# Chapter 3 SQL介绍 Introduction to SQL

SQL语言包含以下几个部分：

1. 数据定义语言 Data Definition Language(DDL)

    - `create table`, `alter table`, `drop table`
    - `create index`, `drop index`
    - `create view`, `drop view`
    - `create trigger`, `drop trigger`

2. 数据操纵语言 Data Manipulation Language(DML)

    - `select...from`
    - `insert`, `delete`, `update`

3. 数据控制语言 Data Control Language(DCL)

    - `grant`, `revoke`

---

## 3.1 数据定义语言 Data Definition Language(DDL)

### 数据类型 Data Types
#### 域类型 Domain Types

| 类型 | 简介 |
| --- | --- |
| **char(n)** | 固定长度字符串，长度为 $n$.(不足的话用空格补足) |
| **varchar(n)** | 可变长度字符串，最大长度为 $n$. |
| **int** | 整数.(范围取决于机器) |
| **smallint** | 小整数.(范围取决于机器) |
| **numeric(p, d)** | 定点数，共 $p$ 位，小数点后 $d$ 位. |
| **real, double precision** | 浮点数，单精度和双精度.(精度取决于机器) |
| **float(n)** | 浮点数，精度为 $n$ 位. |

#### 内置数据类型 Built-in Data Types

| 类型 | 简介 | 示例 |
| --- | --- | --- |
| **date** | 日期类型，包含年（4位），月，日. | `date '2005-7-27'` |
| **time** | 一天中的时间，包含时，分，秒. | `time '09:00:30'`, `time '09:00:30.75'` |
| **timestamp** | 时间戳类型，包含年，月，日，时，分，秒. | `timestamp '2005-7-27 09:00:30.75'` |
| **interval** | 时间间隔.  | `interval '1' day` |

- date/time/timestamp相减会得到interval类型结果，interval类型也可以加到date/time/timestamp上.

---

- date, time functions:
    - **`current_data()`**, **`current_time()`**
    - **`year()`**, **`month()`**, **`day()`**, **`hour()`**, **`minute()`**, **`second()`**
### 表定义 Table Definition

#### 创建表 Create Table

```sql linenums="0"
create table 表名(
    属性名1 数据类型1 [约束条件1],
    属性名2 数据类型2 [约束条件2],
    ...
    [表级约束条件]
);
```

#### 完整性约束 Integrity Constriants

1. **`not null`**: 保证非空.
2. **`default 0`**: 指定缺省值.
3. **`primary key(A_1, ..., A_n)`**: 主键，不重且非空.（若主键只有一个属性，可以直接写在属性定义后面）
4. **`foreign key(A_1, ..., A_n) references r`**: 外键.

    - 可附加 **`on delete/update cascade/set null/restrict/set default`**，即如果被引用的表中主键的某一个值被删除/更改，则当前表中对该值的做法：1. 也删除/更改 2. 置空 3. 禁止删除/更改（回退） 4. 置为缺省值.

5. **`check(P)`**: 断言，保证属性满足条件 $P$.

!!! example
    !!! quote ""
        === "example 1"
            ```sql
            create table instructor(
                ID        char(5),
                name      varchar(20) not null,
                dept_name varchar(20),
                salary    numeric(8, 2)
            )

            1. insert into instructor values('10211', 'Smith', 'Biology', 66000); // success
            2. insert into instructor values('10211', null, 'Biology', 66000); // fail
            ```

        === "example 2"
            声明 $branch\_name$ 为主键，保证 $assets$ 非负的两种等价定义：

            ```sql
            create table branch(
                branch_name char(20) not null,
                branch_city char(30),
                assets      integer,

                primary key(branch_name),
                check(assets >= 0)
            );
            ```

            ```sql
            create table branch(
                branch_name char(20) primary key,
                branch_city char(30),
                assets      integer,

                check(assets >= 0)
            );
            ```
        
#### 删除和更改表 Drop and Alter Table

```sql linenums="0"
drop table 表名; -- 删除表和其中的内容
delete from 表名; -- 删除表中的内容，但保留表

alter table 表名 add 属性名 数据类型 [约束条件]; -- 添加属性，默认初始值为null
alter table 表名 drop 属性名; -- 删除属性（很多数据库不支持此操作）
alter table 表名 modify 属性名 数据类型 [约束条件]; -- 修改属性类型
```

## 3.2 数据操纵语言 Data Manipulation Language(DML)