# Chapter 5 高级SQL Advanced SQL
## 5.1 从编程语言访问SQL Accessing SQL from a Programming Language
原因：

1. 有些查询不能用SQL表达；
2. 为了实现非描述式操作（比如打印、互动等不能直接在SQL中执行）

方式：

1. **动态SQL(Dynamic SQL):** 调用API, 在程序运行时构建生成SQL语句。介绍两种标准：
    1. **JDBC(Java Database Connectivity)** - Java.
    2. **ODBC(Open Database Connectivity)** - C, C++, C#...
2. **嵌入式SQL(Embedded SQL):** 好处是在编译阶段就可以做类型检查。
    1. **Embedded SQL in C**.
    2. **SQLJ** - embedded SQL in Java.
    3. **JPA(Java Persitence API)** - Or mapping of Java.

---

### JDBC
通过 Java 类来封装，分为以下几个步骤：

1. 建立连接 | open a connection;
2. 创建语句对象 | create a "statement" object;
3. 执行查询 | execute queries using the Statement object(send queries and fetch results);
4. 异常机制 | exception mechanism(handle errors).

```java linenums="0"
public static void JDBCexample(String dbid, String userid, String passwd) { 
    try { 
        Connection conn = DriverManager.getConnection(
            "jdbc:oracle:thin:@db.yale.edu:2000:univdb", userid, passwd); 
        Statement stmt = conn.createStatement(); 
            ... Do Actual Work ...
        stmt.close();	
        conn.close();	
    }		
    catch (SQLException sqle) { 		
        System.out.println("SQLException : " + sqle);		
    }		
}

// Update to database
try {
    stmt.executeUpdate(
        "insert into instructor values('77987', 'Kim', 'Physics', 98000)");
} catch (SQLException sqle) {
    System.out.println("Could not insert tuple. " + sqle);
}

// Execute query and fetch and print results
ResultSet rset = stmt.executeQuery(
    "select dept_name, avg (salary)
     from instructor
     group by dept_name");
while (rset.next()) {
    System.out.println(rset.getString("dept_name") + " " +rset.getFloat(2));
}
```

#### JDBC 预处理语句 JDBC Prepared Statements

若直接将用户输入的字符串拼接到SQL语句中，可能会导致**SQL注入式攻击（SQL injection）**。预处理语句可以避免这种情况。

```java linenums="0"
PreparedStatement pStmt = conn.prepareStatement( 
    "insert into account values(?,?,?)");
pStmt.setString(1, "A_9732"); 
pStmt.setString(2, "Perryridge"); 
pStmt.setInt(3, 1200); 
pStmt.executeUpdate(); 
pStmt.setString(1, "A_9733"); 
pStmt.executeUpdate(); 
```

#### JDBC 元数据 JDBC Metadata
调用 `getMetaData()` 可以获得查询结果的元数据，也可以获得数据库的元数据。

```java linenums="0"
ResultSetMetaData rsmd = rs.getMetaData();
for(int i = 1; i <= rsmd.getColumnCount(); i++) {
    System.out.println(rsmd.getColumnName(i));
    System.out.println(rsmd.getColumnTypeName(i));
}

DatabaseMetaData dbmd = conn.getMetaData();
ResultSet rs = dbmd.getColumns(null, "univdb", "department", "%");
while( rs.next()) {
    System.out.println(rs.getString("COLUMN NAME"), rs.getString("TYPE NAME"));
}
```

#### JDBC 事务控制 JDBC Transaction Control

可调用函数 `setAutoCommit()`, `commit()`, `rollback()` 来控制事务。

### ODBC

不同于JDBC, ODBC 以函数形式提供接口，基本流程如下：

1. ODBC 初始化
    - `SQLAllocEnv()`, `SQLAllocConnect()`, `SQLConnect()`, `SQLAllocStmt()`
2. 执行 SQL 命令
    - `SQLExecDirect()`, `SQLPrepare()`, `SQLExecute()`
3. 获取结果数据
    - `SQLFetch()`, `SQLGetData()`
4. 释放空间
    - `SQLFreeStmt()`, `SQLDisConnect()`, `SQLFreeConnect()`, `SQLFreeEnv()`
  
```java linenums="0"
void ODBCexample()
{
    RETCODE error;
    HENV env; /* environment */
    HDBC conn; /* database connection */

    SQLAllocEnv(&env); // 分配环境句柄
    SQLAllocConnect(env, &conn); // 分配连接句柄
    SQLConnect(conn, "db.yale.edu", SQL_NTS, "avi", SQL_NTS, "avipasswd", SQL_NTS); // 连接数据源
    {
        char deptname[80];
        float salary;
        int lenOut1, lenOut2;
        HSTMT stmt;

        char * sqlquery = "select dept name, sum (salary)
                           from instructor
                           group by dept name";
        SQLAllocStmt(conn, &stmt); // 分配语句句柄
        error = SQLExecDirect(stmt, sqlquery, SQL_NTS); // 直接执行SQL语句
        if (error == SQL_SUCCESS) {
            SQLBindCol(stmt, 1, SQL_C_CHAR, deptname, 80, &lenOut1); // 自定义变量绑定
            SQLBindCol(stmt, 2, SQL_C_FLOAT, &salary, 0, &lenOut2);
            while (SQLFetch(stmt) == SQL_SUCCESS) { // 获取新的一行
                printf (" %s %g∖n", deptname, salary);
            }
        }
        SQLFreeStmt(stmt, SQL_DROP); // 释放语句句柄
    }
    SQLDisconnect(conn); // 断开数据源连接
    SQLFreeConnect(conn); // 释放连接句柄
    SQLFreeEnv(env); // 释放环境句柄
}
```

- 其中 `SQL_NTS` 表示 Null-Terminated String(结束符), 当字符串以 NULL 结束时，可用它表示字符串长度。

#### ODBC 预处理语句 ODBC Prepared Statements

可调用 `SQLPrepare()`, `SQLBindParameter()`, `SQLExecute()` 来执行预处理语句。

#### ODBC 事务控制 ODBC Transaction Control

可调用 `SQLSetConnectOption()`, `SQLTransact()` 来控制事务。

### Embedded SQL in C

在C语言中，使用 **`EXEC SQL`** 来标识SQL语句。

- 宿主语言的变量可以在 SQL 语句中使用，使用时在变量前加 **`:`**.

#### 单行操作 Single Record Operations

!!! quote ""
    === "insert"
        ```C linenums="0"
        main() {
            EXEC SQL INCLUDE SQLCA; // 存放语句执行状态的数据结构
            EXEC SQL BEGIN DECLARE SECTION;
                char account_no[11]; // 以下为宿主变量声明
                char branch_name[16];
                int balance;
            EXEC SQL END DECLARE SECTION; // 声明段结束
            EXEC SQL CONNECT TO bank_db USER Adam USING Eve;

            scanf("%s %s %d", account_no, branch_name, &balance);
            EXEC SQL INSERT INTO account VALUES(:account_no, :branch_name, :balance);
            if(SQLCA.sqlcode != 0) printf("Error!\n");
            else printf("Success!\n");
        }
        ```
    === "delete"
        ```C linenums="0"
        main() {
            EXEC SQL INCLUDE SQLCA; // 存放语句执行状态的数据结构
            EXEC SQL BEGIN DECLARE SECTION;
                char account_no[11]; // 宿主变量声明
            EXEC SQL END DECLARE SECTION; // 声明段结束
            EXEC SQL CONNECT TO bank_db USER Adam USING Eve;

            scanf("%s", account_no);
            EXEC SQL DELETE FROM account WHERE account_number = :account_no;
            if(SQLCA.sqlcode != 0) printf("Error!\n");
            else printf("Success!\n");
        }
        ```
    === "update"
        ```C linenums="0"
        main() {
            EXEC SQL INCLUDE SQLCA; // 存放语句执行状态的数据结构
            EXEC SQL BEGIN DECLARE SECTION;
                char account_no[11]; // 宿主变量声明
                int balance;
            EXEC SQL END DECLARE SECTION; // 声明段结束
            EXEC SQL CONNECT TO bank_db USER Adam USING Eve;

            scanf("%s %d", account_no, &balance);
            EXEC SQL UPDATE account SET balance = balance + :balance WHERE account_number = :account_no;
            if(SQLCA.sqlcode != 0) printf("Error!\n");
            else printf("Success!\n");
        }
        ```
    === "select(single record)"
        ```C linenums="0"
        main() {
            EXEC SQL INCLUDE SQLCA; // 存放语句执行状态的数据结构
            EXEC SQL BEGIN DECLARE SECTION;
                char account_no[11]; // 宿主变量声明
                int balance;
                int mask; // 指示变量，=0 正常, <0 NULL, >0 截断 
            EXEC SQL END DECLARE SECTION; // 声明段结束
            EXEC SQL CONNECT TO bank_db USER Adam USING Eve;

            scanf("%s %d", account_no, &balance);
            EXEC SQL SELECT balance INTO :balance:mask FROM account WHERE account_number = :account_no;
            if(SQLCA.sqlcode != 0) printf("Error!\n");
            else if(mask == 0) printf("balance = %d\n", balance);
            else printf("balance is NULL!\n");
        }
        ```

#### 多行操作 Multiple Records Operations

使用游标 **`cursor`** 来处理多行操作。

!!! quote ""
    === "select(multiple records)"
        ```C linenums="0"
        main() {
            EXEC SQL INCLUDE SQLCA;
            EXEC SQL BEGIN DECLARE SECTION;
                char customer_name[21];
                char account_no[11];
                int balance;
            EXEC SQL END DECLARE SECTION;
            EXEC SQL CONNECT TO univ_db USER Adam USING Eve;

            EXEC SQL DECLARE account_cursor CURSOR FOR // 声明游标
                SELECT account_number, balance
                FROM depositor natural join account
                WHERE depositor.customer_name = :customer_name;
            scanf("%s", customer_name);
            EXEC SQL OPEN account_cursor; // 执行查询
            for(; ; ) {
                EXEC SQL FETCH account_cursor INTO :account_no, :balance; // 获取新的一行
                if(SQLCA.sqlcode != 0) break;
                printf("%s %d\n", account_no, balance);
            }
            EXEC SQL CLOSE account_cursor; // 删除本次查询的结果
        }
        ```
    === "delete or update(current record)"
        - 使用 **`where current of cursor_name`** 来指示当前游标所指向的记录。

        ```C linenums="0"
        // continue...
        scanf("%s", customer_name);
        EXEC SQL OPEN account_cursor;
        for(; ; ) {
            EXEC SQL FETCH account_cursor INTO :account_no, :balance;
            if(SQLCA.sqlcode != 0) break;
            if(balance < 1000) {
                EXEC SQL UPDATE account SET balance = balance * 1.05 WHERE current of account_cursor;
            }
            else {
                EXEC SQL UPDATE account SET balance = balance * 1.06 WHERE current of account_cursor;
            }
        }
        ```
#### 动态语句 Dynamic Statements

Embedded SQL 也支持动态语句，同样是运行时调用函数再进行检查。

```C linenums="0"
// delete 直接执行
EXEC SQL BEGIN DECLARE SECTION;
    char statement[256];
EXEC SQL END DECLARE SECTION;
char table_name[32], condition[128];
gets(table_name);
gets(condition);
if(strlen(condition) > 0) {
    sprintf(statement, "DELETE FROM %s WHERE %s", table_name, condition);
}
else {
    sprintf(statement, "DELETE FROM %s", table_name);
}
EXEC SQL EXECUTE IMMEDIATE :statement;

// update 使用预处理语句
gets(table_name);
gets(update_column);
gets(search_column);
sprintf(statement, "UPDATE %s SET %s=? WHERE %s=?", table_name, update_column, search_column);
EXEC SQL PREPARE mystmt FROM :statement;
for(; ; ) {
    scanf("%d %d", &new_value, &search_value);
    EXEC SQL EXECUTE mystmt USING :new_value, :search_value;
}
```

## 5.2 SQL函数与过程 Functions and Procedures in SQL