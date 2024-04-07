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
    create index studentID_index on student(ID)
    ```
    以下语句效率提高：
    ```sql linenums="0"
    select *
    from student
    where ID = '12345';
    ```

### 断言 Assertion
每次对数据库的修改都会被检查，如果不符合断言，将报错。
```sql linenums="0"
create assertion 断言名 check 条件
```
!!! warning
    然而现有的大部分 DBMS 都不支持断言。

### 触发 Trigger
当数据库中的某个事件发生时，触发器会自动执行一个动作。

**ECA Rule:** Event(insert, delete, update) - Condition - Action.

#### 行级触发器 Row Level Triggers

```sql linenums="0"
create trigger 触发器名 before/after insert/delete/update on 表名
for each row
when 条件
begin
    ...
end
```

- 可以指定表的属性，如 `after update of takes on grade`.
- 可以访问操作前后的属性值，用 `referencing old row as`, `referencing new row as`.

??? example
    !!! quote ""
        === "example 1"
            利用日志 `account_log(account, amount, datetime)`，记录大额交易。

            ```sql linenums="0"
            create trigger account_trigger after update on account(balance)
            referencing new row as nrow
            referencing old row as orow
            for each row
            when nrow.balance - orow.balance > 200000 or
                orow.balance - nrow.balance > 50000
            begin
                insert into account_log values(nrow.account_number, nrow.balance - orow.balance, current_time())
            end;
            ```

        === "example 2"
            实现引用完整性。（也可以用断言，但断言是全局性的，代价比较高）
            
            如 $time\_slot\_id$ 不是表 $timeslot$ 的主键，所以我们不能直接定义一个从表 $section$ 到表 $timeslot$ 的外键约束。

            ```sql linenums="0"
            create trigger timeslot_check1 after insert on section
            referencing new row as nrow
            for each row
            when (nrow.time_slot_id not in(
                select time_slot_id
                from time_slot
            ))
            begin
                rollback
            end;

            create trigger timeslot_check2 after update on timeslot
            referencing old row as orow
            for each row
            when(orow.time_slot_id not in(
                select time_slot_id
                from time_slot -- 被修改的元组的属性值
            ) and (orow.time_slot_id in(
                select time_slot_id
                from section) -- 仍然在section中被引用，说明不合法
            ))
            begin
                rollback
            end;
            ```

#### 语句级触发器 Statement Level Triggers

- 使用 `for each statement` 替换 `for each row`;
- 使用 `referencing old table`, `referencing new table` 替换 `referencing old row`, `referencing new row`.

??? example
    !!! quote ""
        === "example 1"
            保证平均分及格。

            ```sql linenums="0"
            create trigger score_trigger after update on register(score)
            referencing new table as ntable
            for each statement
            when some(select average(score)
                      from ntable
                      group by cno) < 60
            begin
                rollback
            end;
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

### 授权 Authorization

#### 授权的形式 Forms of Authorization

|数据库内容操作|简介|
|---|---|
|**select**| 允许读取数据，不允许修改数据。|
|**insert**| 允许插入新数据，不允许修改数据。|
|**update**| 允许修改数据，不允许删除数据。|
|**delete**| 允许删除数据。|

|数据库模式操作|简介|
|---|---|
|**index**| 允许创建和删除索引。|
|**resources**| 允许创建表。|
|**alteration**| 允许增加和删除（修改）表的属性。|
|**drop**| 允许删除表。|

#### 授权与视图 Authorization and Views

- 用户可以被给予对视图的访问权限，而不具有对基表的访问权限。
- 创建视图不需要对基表有 resources 权限。（因为创建视图并没有创建新表）
- 视图的创建者（当然如果没有对基表的访问权限，则无法创建视图）对视图的权限不会超过对基表的权限。

#### 授予权限 Grant

```sql linenums="0"
grant 权限列表 on 表名/视图名 to 用户名列表
```

- 其中的“用户名”可以是：
    1. 具体的用户名（user-id）；
    2. `public`，表示所有用户；
    3. 一个角色（role）。

!!! example
    ```sql linenums="0"
    grant select on instructor to U_1, U_2, U_3;
    grant insert, update on department to public;
    grant update(budget) on department to U_1, U_2;
    grant reference(dept_name) on department to Mariano;
    grant all privileges on department to U_1;
    ```

#### 撤销权限 Revoke

```sql linenums="0"
revoke 权限列表 on 表名/视图名 from 用户名列表
```

- 如果同样的权限被授予多次，那么撤销时只撤销一次。

!!! example
    ```sql linenums="0"
    revoke select on branch from U_1, U_2, U_3;
    ```

#### 传递权限 Transfer of Privileges
```sql linenums="0"
grant select on department to Amit with grant option;
revoke select on department from Amit, Satoshi cascade;
revoke select on department from Amit, Satoshi restrict;
```

- **`with grant option`**：允许用户将权限传递给其他用户。
- 撤销权限时，**`cascade`** 自动把后继权限全部收回，**`restrict`**：不收回。
- **`revoke grant option`** 收回传递权限的权限。

#### 角色 Roles
```sql linenums="0"
create role 角色名
grant 角色名 to 用户名列表
```

!!! example 
    ```sql linenums="0"
    -- 权限可以被授予角色
    grant select on takes to instructor;
    -- 角色可以被授予用户，也可以被授予其他角色
    create role teaching_assistant;
    grant teaching_assistant to instructor;
    -- chain of roles
    create role dean;
    grant instructor to dean;
    grant dean to Satoshi;
    ```

#### 其它授权语句 Other Authorization Statements

### 审计跟踪 Audit Trails

