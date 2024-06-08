# SQL 预编译语句的使用与局限


使用预编译语句 (Prepared Statements) 将编译好的语句存储在数据库, 需要使用时传递参数即可, 可以提高查询性能并防止 SQL 注入。

## 使用
使用 mysql8.4.0, 表如下:
```
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(50) NOT NULL
);
```

### 直接在 mysql 中
创建预编译语句:
```sql
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?';
```

设置参数:
```sql
SET @user_id = 1;
```

执行:
```sql
EXECUTE stmt USING @user_id;
```

执行结果:
```
+----+----------+-------------+
| id | username | password    |
+----+----------+-------------+
| 1  | admin    | password123 |
+----+----------+-------------+
```

### python + mysql-connector
```python
from mysql import connector

# 获取连接
connection = connector.connect(
    host="172.17.0.2",
    port="3306",
    user="root",
    password="123456",
    database="test",
)

# 获取游标, "prepared=True" 用于开启预编译
cursor = connection.cursor(prepared=True)

# 编写语句, 其中 "%s" 与 python 中格式化字符串的 "%s" 不同, 仅表示此处需要一个参数, 可以用 "?" 代替
sql = "select * from users where id = %s"

# 执行
cursor.execute(sql, (1,))

print(cursor.fetchall())
```

执行结果:
```
[(1, 'admin', 'password123')]
```

### java + jdbc
```java
// ...

// 获取连接
Connection conn = JDBCUtil.getConnection();

// # 编写语句
String sql = "select * from users where id = ?";

// 预编译
PreparedStatement pstam = conn.prepareStatement(sql);

// 设置参数
pstam.setInt(1, 1);

// 执行
int num = pstam.executeSelect();

// ...
```

## 局限
预编译语句不能将表名或者列名设置为参数。

将表名设置为参数会报错:
```sql
> PREPARE stmt FROM 'SELECT * FROM ? WHERE id = 1';

(1064, "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '? WHERE id = 1' at line 1")
```

将列名设置为参数无法得到结果:
```sql
> PREPARE stmt FROM 'SELECT * FROM users WHERE ? = 1';
> SET @p1 = 'id';
> EXECUTE stmt USING @p1;

+----+----------+----------+
| id | username | password |
+----+----------+----------+
+----+----------+----------+
```
