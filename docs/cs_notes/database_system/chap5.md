# Chapter 5 高级SQL Advanced SQL
## 5.1 从编程语言访问SQL Accessing SQL from a Programming Language
原因：

1. 有些查询不能用SQL表达；
2. 为了实现非描述式操作（比如打印、互动等不能直接在SQL中执行）

方式：

1. **动态SQL(Dynamic SQL):** 调用API, 在程序运行时构建生成SQL语句。介绍两种标准：
    1. **JDBC(Java Database Connectivity)** - Java.
    2. **ODBC(Open Database Connectivity)** - C, C++, C#...
2. **嵌入式SQL(Embedded SQL):** 介绍两种标准：
    1. **SQLJ** - embedded SQL in Java.
    2. **JPA(Java Persitence API)** - Or mapping of Java.

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

#### 预处理语句 Prepared Statements

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

#### 元数据 Metadata
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

#### 事务控制 Transaction Control

可调用函数 `setAutoCommit()`, `commit()`, `rollback()` 来控制事务。

### ODBJ

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

    SQLAllocEnv(&env);
    SQLAllocConnect(env, &conn);
    SQLConnect(conn, "db.yale.edu", SQL NTS, "avi", SQL NTS, "avipasswd", SQL NTS);
    {
        char deptname[80];
        float salary;
        int lenOut1, lenOut2;
        HSTMT stmt;

        char * sqlquery = "select dept name, sum (salary)
                           from instructor
                           group by dept name";
        SQLAllocStmt(conn, &stmt);
        error = SQLExecDirect(stmt, sqlquery, SQL NTS);
        if (error == SQL SUCCESS) {
            SQLBindCol(stmt, 1, SQL C CHAR, deptname , 80, &lenOut1);
            SQLBindCol(stmt, 2, SQL C FLOAT, &salary, 0 , &lenOut2);
            while (SQLFetch(stmt) == SQL SUCCESS) {
                printf (" %s %g∖n", deptname, salary);
            }
        }
        SQLFreeStmt(stmt, SQL DROP);
    }
    SQLDisconnect(conn);
    SQLFreeConnect(conn);
    SQLFreeEnv(env);
}
```

### SQLJ

### Embedded SQL