# Chapter 4 中级SQL Intermediate SQL

## 4.1 数据定义语言 Data Definition Language(DDL) - Part 2

### 视图 View

**视图（view）**不是存储在数据库中的表，而是一个虚拟表，即保存了一个 查询的表达。

#### 创建视图 Create View
```sql linenums="0"
create view 视图名 as 查询语句
create view 视图名(属性名1, 属性名2, ...) as 查询语句 -- 可重命名属性

-- view的删除同table
drop view 视图名
```

#### 视图的更新 Update of a View 

视图是虚表，对其进行的所有操作都将转化为对基表的操作。 

!!! example
    在视图faculty中添加一个新元组。
    相当于往表instructor中插入了一行，窗口中看不到的属性默认置为**null**.

    ```sql linenums="0"
    create view faculty as
        select ID, name, dept_name
        from instructor
    insert into faculty values('30765', 'Green', 'Music');
    ```

!!! warning 
    视图的更新可能涉及多个表，且可能无法唯一转换为真实表的更新操作。

    例如：
    ```sql linenums="0"
    create view instructor_info as
        select ID, name, building
        from instructor, department
        where instructor.dept_name = department.dept_name;
    insert into instructor_info values('69987', 'White', 'Taylor');
    ```
    如果有多个 $department$ 中的 $building$ 属性为 `Taylor`，那么这个插入操作就会失败。

    所以大部分SQL系统只允许对简单的视图（updatable views）进行更新:

    - from子句中只有一个表.
    - select子句只包含表中的属性名，且未被包含的属性名都允许置null.
    - 不含group by或者having子句.

#### 视图和逻辑独立性 View and Logical Data Independence
以下例子展示了使用视图的好处。

**将关系 $S(a,\ b,\ c)$ 分裂为两个子关系 $S1(a,\ b)$ 和 $S2(a,\ c)$ 并维持逻辑独立性。**
```sql linenums="0"
create table S1 ...;
create table S2 ...;
insert into S1 select a, b from S;
insert into S2 select a, c from S;
drop table S;
create view S(a, b, c) as select a, b, c from S1 natural join S2;

-- 操作示例
select * from S where ... => select * from S1 natural join S2 where ...
insert into S values(1, 2, 3) => insert into S1 values(1, 2); insert into S2 values(1, 3);
```

### 索引 Indexes

建立一个B+树索引，可以加速查询操作。

逻辑上写法不变，但物理层面效率提高。

!!! example

    ```sql linenums="0"
    create studentID_index on student(ID)
    ```
    以下语句效率提高：
    ```sql linenums="0"
    select *
    from student
    where ID = '12345';
    ```

## 4.2 数据操纵语言 Data Manipulation Language(DML) - Part 2

### 事务 Transactions

**事务（Transactions）**是数据库中的一个逻辑工作单元，包含一系列的操作。

- 每个事务自动开始，以 `commit work` 或 `rollback work` 结束。

!!! example
    ```sql linenums="0"
    set autocommit=0 -- 关闭自动提交
    
    update account set balance = balance - 100 where ano = '1001';
    update account set balance = balance + 100 where ano = '1002';
    commit;

    update account set balance = balance - 200 where ano = '1003';
    update account set balance = balance + 200 where ano = '1004';
    commit;

    update account set balance = balance + balance * 2.5%;
    commit;
    ```

#### 事务的ACID特性 ACID Properties of Transactions

- **原子性（Atomicity）**：事务中的所有操作要么全部执行，要么全部不执行。
- **一致性（Consistency）**：事务执行前后，数据库的状态应保持一致。
- **隔离性（Isolation）**：事务的执行不受其他事务的影响。
- **持久性（Durability）**：事务执行后，对数据库的改变应该是永久的。

??? question "缓存区存了一半突然断电怎么办？"
    需要日志，数据库断电后可通过日志恢复数据。

    因为数据库的不同块不储存在一起，写的时候比较慢，而日志写在顺序的一整块空间中，写起来快。

## 4.3 数据控制语言 Data Control Language(DCL)