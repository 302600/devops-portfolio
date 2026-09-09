# MySQL 学习详解 —— 从零开始，像讲故事一样学数据库

> 这篇文档的目标：让一个完全不懂数据库的人，看完之后能理解MySQL是什么、怎么用、怎么管。
> 我会像讲故事一样，一步一步来，每个概念都用生活中的例子解释。
> 每个语法都会详细讲解，你看完就能照着敲。

---

# 第一章：MySQL是什么？用最简单的话告诉你

## 1.1 一个生活比喻

想象你开了一家小卖部。

你有很多商品：可乐、薯片、方便面……每件商品有名称、价格、库存数量。

你把这些信息记在哪里？

- 记在脑子里？记不住。
- 记在纸上？改起来麻烦，找起来也慢。
- 记在Excel表格里？能用了，但如果数据有一百万条呢？Excel就卡死了。

**MySQL就是一张超级强大的、电子版的表格存放系统。**

它能存放海量的数据（几千万条都不卡），而且你用一种叫做"SQL"的语言跟它说话，让它帮你存数据、找数据、改数据、删数据。

## 1.2 数据库、表、数据的关系

还是用小卖部比喻：

```
数据库（Database）= 一个仓库
  └── 表（Table）= 仓库里的货架
        └── 数据（Data）= 货架上的商品
```

- **数据库**：就像一个独立的仓库，里面可以放很多东西。
- **表**：就像仓库里的货架，每个货架放一类东西。比如"商品表"放所有商品信息，"员工表"放所有员工信息。
- **数据**：货架上的每一件商品，就是一条数据。

一张表长这样：

| id | name | price | stock |
|----|------|-------|-------|
| 1  | 可乐 | 3.5   | 100   |
| 2  | 薯片 | 7.0   | 50    |
| 3  | 方便面 | 2.5 | 200   |

- 每一**行**（横着看）叫一条"记录"，比如"可乐 3.5元 100件"就是一条记录。
- 每一**列**（竖着看）叫一个"字段"，比如"name"这一列存的都是商品名字。

## 1.3 SQL是什么

SQL（读作"S-Q-L"或"sequel"）就是跟MySQL说话的语言。

你用SQL告诉MySQL：
- "帮我存一条数据" → INSERT
- "帮我找数据" → SELECT
- "帮我改数据" → UPDATE
- "帮我删数据" → DELETE

这四个操作简称 **CRUD**（Create创建、Read读取、Update更新、Delete删除），是数据库最基础的操作。

## 1.4 MySQL怎么装到你电脑上的

MySQL是一个软件，装在服务器（或你的虚拟机）上。

安装方式有两种：
1. **yum安装**：用Linux的包管理器自动下载安装，简单省事。
2. **源码编译安装**：下载源代码自己编译，可以自定义功能，但麻烦。

装完之后，MySQL会作为一个"服务"在后台运行，你随时可以用客户端连接它。

## 1.5 怎么连接MySQL

MySQL装好后，你需要"登录"才能使用。

```bash
mysql -u root -p
```

逐个解释：
- `mysql` → 启动MySQL客户端程序
- `-u root` → u是user（用户）的缩写，用root这个用户登录
- `-p` → p是password（密码）的缩写，提示你输入密码

输入密码后看到 `mysql>` 就说明登录成功了，接下来就可以输入SQL语句了。

**每条SQL语句结尾必须加分号 `;`**，就像每句话末尾加句号一样。不加的话MySQL会以为你还没说完，一直等着你。

---

# 第二章：数据库操作（仓库级别）

## 2.1 查看所有数据库

```sql
SHOW DATABASES;
```

翻译："把所有仓库的名字列出来给我看看。"

执行后会显示类似：

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
```

前四个是MySQL自带的系统数据库，别动它们。`testdb`是你自己创建的。

## 2.2 创建数据库

```sql
CREATE DATABASE testdb;
```

逐词翻译：
- `CREATE` → 创建
- `DATABASE` → 数据库
- `testdb` → 数据库的名字（自己起，随便叫什么都行）
- `;` → 语句结束

翻译成人话："创建一个名叫testdb的数据库。"

### 完整语法

```sql
CREATE DATABASE [IF NOT EXISTS] 数据库名 [CHARACTER SET 字符集] [COLLATE 排序规则];
```

中括号 `[]` 里的内容是可选的，不写也行。

- `IF NOT EXISTS` → "如果不存在才创建"，防止重复创建报错
- `CHARACTER SET` → 设定字符集，比如 utf8mb4（支持中文和emoji表情）
- `COLLATE` → 排序规则，比如 utf8mb4_general_ci

**推荐写法：**

```sql
CREATE DATABASE IF NOT EXISTS testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

翻译："如果testdb不存在就创建它，字符集用utf8mb4，排序规则用utf8mb4_general_ci。"

### utf8 vs utf8mb4 是什么区别？

- `utf8` → 最多存3个字节的字符，存中文够用，但存不了emoji表情（emoji是4字节）
- `utf8mb4` → 最多存4个字节的字符，中文、emoji都能存

**永远用utf8mb4，不会有问题。**

## 2.3 进入数据库

```sql
USE testdb;
```

翻译："我要进入testdb这个仓库，接下来的操作都在这里面。"

**这一步很重要！** 就像你要从货架上拿东西，得先走进对应的仓库。不执行USE，MySQL不知道你要在哪个库里操作。

## 2.4 删除数据库

```sql
DROP DATABASE testdb;
```

翻译："把testdb这个仓库整个删掉，里面的所有表和数据都没了。"

**警告：DROP DATABASE是删库操作，数据不可恢复！** 生产环境绝对不能随便敲。网上那个段子"从删库到跑路"说的就是这个操作。

### 完整语法

```sql
DROP DATABASE [IF EXISTS] 数据库名;
```

- `IF EXISTS` → "如果存在才删除"，防止删一个不存在的库报错

---

# 第三章：表操作（货架级别）

## 3.1 创建表

建表就像打造一个货架，你要规定这个货架有几层、每层放什么。

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT,
    email VARCHAR(100)
);
```

逐行解释：

### 第一行：CREATE TABLE users

- `CREATE TABLE` → 创建表
- `users` → 表名（自己起名字）
- `(` → 开始定义表的列

### 第二行：id INT PRIMARY KEY AUTO_INCREMENT

- `id` → 列名，叫id（编号的意思）
- `INT` → 数据类型，INT就是整数（Integer的缩写），说明这列只能存数字
- `PRIMARY KEY` → 主键，意思是"这一列是每条记录的唯一标识，不能重复，不能为空"。就像每个人的身份证号，不能重复。
- `AUTO_INCREMENT` → 自动增长，每插入一条新数据，id自动加1（1, 2, 3, 4...），不用你手动指定

### 第三行：name VARCHAR(50) NOT NULL

- `name` → 列名，名字
- `VARCHAR(50)` → 数据类型，VARCHAR是"可变长度字符串"，50表示最多50个字符。存"张三"只占2个字符的空间，存"张三丰"占3个字符的空间，不会浪费。
- `NOT NULL` → 不能为空，必须有值。你不能插入一条没有名字的记录。

### 第四行：age INT

- `age` → 列名，年龄
- `INT` → 整数类型
- 没有NOT NULL，说明这列可以为空（有些人不想透露年龄）

### 第五行：email VARCHAR(100)

- `email` → 列名，邮箱
- `VARCHAR(100)` → 最多100个字符的字符串
- 可以为空

### 最后一行：);

- `)` → 结束列定义
- `;` → 语句结束

## 3.2 常用数据类型一览

建表时你需要告诉MySQL每一列存什么类型的数据，就像货架的每一层标明了"这层放饮料、那层放零食"。

### 整数类型

| 类型 | 范围 | 用途 |
|------|------|------|
| TINYINT | -128 到 127 | 存小数字，比如性别（0女1男） |
| INT | -21亿 到 21亿 | 存普通数字，比如年龄、数量 |
| BIGINT | 非常大 | 存大数字，比如订单号 |

### 小数类型

| 类型 | 用途 |
|------|------|
| DECIMAL(10,2) | 存钱，10位总长度，2位小数。比如99999999.99 |
| FLOAT | 存普通小数，精度低 |
| DOUBLE | 存普通小数，精度高 |

**存钱永远用DECIMAL，不用FLOAT/DOUBLE，因为浮点数有精度问题。**

### 字符串类型

| 类型 | 用途 |
|------|------|
| CHAR(10) | 固定长度10，存不够用空格补齐。比如存固定长度的编号 |
| VARCHAR(255) | 可变长度，最多255字符。最常用 |
| TEXT | 存长文本，比如文章内容 |
| LONGTEXT | 存超长文本，最多4GB |

**CHAR vs VARCHAR的区别：**

```
CHAR(10) 存 "abc"  → 实际占 10 个字符空间（后面补7个空格）
VARCHAR(10) 存 "abc" → 实际占 3 个字符空间 + 1个长度标识
```

**大部分情况用VARCHAR，只有固定长度的数据（如身份证18位、手机号11位）才用CHAR。**

### 日期时间类型

| 类型 | 格式 | 用途 |
|------|------|------|
| DATE | 2026-06-28 | 只存日期 |
| TIME | 14:30:00 | 只存时间 |
| DATETIME | 2026-06-28 14:30:00 | 存日期+时间 |
| TIMESTAMP | 2026-06-28 14:30:00 | 时间戳，自动记录修改时间 |

## 3.3 查看表结构

```sql
DESC users;
```

翻译："给我看看users这个表的结构，有哪些列、什么类型。"

`DESC` 是 `DESCRIBE`（描述）的缩写。

执行后会显示：

```
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int          | NO   | PRI | NULL    | auto_increment |
| name  | varchar(50)  | NO   |     | NULL    |                |
| age   | int          | YES  |     | NULL    |                |
| email | varchar(100) | YES  |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
```

各列含义：
- `Field` → 列名
- `Type` → 数据类型
- `Null` → 是否允许为空（NO=不允许，YES=允许）
- `Key` → 是否是键（PRI=主键）
- `Default` → 默认值
- `Extra` → 额外信息（auto_increment=自动增长）

## 3.4 查看所有表

```sql
SHOW TABLES;
```

翻译："把当前数据库里所有的表列出来。"

## 3.5 删除表

```sql
DROP TABLE users;
```

翻译："把users这个表整个删掉，表里的数据也没了。"

### 完整语法

```sql
DROP TABLE [IF EXISTS] 表名;
```

## 3.6 修改表结构

表建好了之后还能改，比如加列、删列、改列类型。

### 加一列

```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

翻译："给users表加一列叫phone，类型是VARCHAR(20)。"

- `ALTER TABLE` → 修改表
- `ADD COLUMN` → 加一列

### 删一列

```sql
ALTER TABLE users DROP COLUMN phone;
```

翻译："把users表的phone列删掉。"

### 改列类型

```sql
ALTER TABLE users MODIFY COLUMN age TINYINT;
```

翻译："把age列的类型改成TINYINT。"

### 改列名

```sql
ALTER TABLE users CHANGE COLUMN age user_age INT;
```

翻译："把age列改名叫user_age，类型是INT。"

### 完整语法总结

```sql
-- 加列
ALTER TABLE 表名 ADD COLUMN 列名 数据类型 [约束];

-- 删列
ALTER TABLE 表名 DROP COLUMN 列名;

-- 改类型
ALTER TABLE 表名 MODIFY COLUMN 列名 新数据类型;

-- 改列名+类型
ALTER TABLE 表名 CHANGE COLUMN 旧列名 新列名 新数据类型;
```

---

# 第四章：数据增删改查（CRUD）—— 最核心的操作

## 4.1 插入数据（INSERT）

### 基本语法

```sql
INSERT INTO 表名 (列名1, 列名2, 列名3) VALUES (值1, 值2, 值3);
```

翻译："往表里插入一条数据，列名1的值是值1，列名2的值是值2……"

### 实际例子

```sql
INSERT INTO users (name, age, email) VALUES ('张三', 25, 'zhangsan@163.com');
```

逐词解释：
- `INSERT INTO users` → 往users表里插入
- `(name, age, email)` → 要填的列是name、age、email
- `VALUES` → 值是
- `('张三', 25, 'zhangsan@163.com')` → 对应的值：name='张三'，age=25，email='zhangsan@163.com'

**注意：id列没写，因为它有AUTO_INCREMENT，会自动生成1。**

**字符串和日期必须用单引号 `' '` 包起来，数字不用。**

### 插入多条数据

```sql
INSERT INTO users (name, age, email) VALUES 
('张三', 25, 'zhangsan@163.com'),
('李四', 30, 'lisi@163.com'),
('王五', 28, 'wangwu@163.com');
```

一条INSERT语句可以插入多条记录，用逗号分隔。

### 插入所有列

```sql
INSERT INTO users VALUES (1, '张三', 25, 'zhangsan@163.com');
```

不写列名时，必须按表结构的顺序给所有列都提供值。**不推荐这种写法**，因为如果表结构变了就会出错。

## 4.2 查询数据（SELECT）

SELECT是SQL里用得最多的操作，就像你在仓库里找东西。

### 查所有列所有行

```sql
SELECT * FROM users;
```

翻译："把users表里所有数据都给我看。"

- `SELECT` → 查询
- `*` → 星号代表"所有列"
- `FROM users` → 从users表

执行结果：

```
+----+------+-----+--------------------+
| id | name | age | email              |
+----+------+-----+--------------------+
|  1 | 张三 |  25 | zhangsan@163.com   |
|  2 | 李四 |  30 | lisi@163.com       |
|  3 | 王五 |  28 | wangwu@163.com     |
+----+------+-----+--------------------+
```

### 查指定列

```sql
SELECT name, age FROM users;
```

翻译："我只要看name和age这两列，其他的不看。"

```
+------+-----+
| name | age |
+------+-----+
| 张三 |  25 |
| 李四 |  30 |
| 王五 |  28 |
+------+-----+
```

### 条件查询（WHERE）

```sql
SELECT * FROM users WHERE age > 26;
```

翻译："把users表里age大于26的记录找出来。"

- `WHERE` → 条件
- `age > 26` → age列的值大于26

结果：

```
+----+------+-----+------------------+
| id | name | age | email            |
+----+------+-----+------------------+
|  2 | 李四 |  30 | lisi@163.com     |
|  3 | 王五 |  28 | wangwu@163.com   |
+----+------+-----+------------------+
```

### WHERE的多种条件

#### 等于

```sql
SELECT * FROM users WHERE name = '张三';
```

找名字等于"张三"的记录。**字符串要用单引号。**

#### 不等于

```sql
SELECT * FROM users WHERE age != 30;
-- 或者
SELECT * FROM users WHERE age <> 30;
```

`!=` 和 `<>` 都是不等于的意思。

#### 大于、小于、大于等于、小于等于

```sql
SELECT * FROM users WHERE age > 25;     -- 大于25
SELECT * FROM users WHERE age < 30;     -- 小于30
SELECT * FROM users WHERE age >= 25;    -- 大于等于25
SELECT * FROM users WHERE age <= 30;    -- 小于等于30
```

#### 多个条件（AND / OR）

```sql
-- 年龄大于25 且 名字是张三
SELECT * FROM users WHERE age > 25 AND name = '张三';

-- 年龄小于26 或 年龄大于28
SELECT * FROM users WHERE age < 26 OR age > 28;
```

- `AND` → 并且（两个条件同时满足）
- `OR` → 或者（满足任意一个条件就行）

#### 在某个范围内（IN）

```sql
SELECT * FROM users WHERE age IN (25, 30);
```

找年龄是25或30的记录。等价于：

```sql
SELECT * FROM users WHERE age = 25 OR age = 30;
```

#### 在区间内（BETWEEN）

```sql
SELECT * FROM users WHERE age BETWEEN 25 AND 30;
```

找年龄在25到30之间（包含25和30）的记录。

#### 模糊匹配（LIKE）

```sql
-- 找名字以"张"开头的
SELECT * FROM users WHERE name LIKE '张%';

-- 找名字包含"三"的
SELECT * FROM users WHERE name LIKE '%三%';

-- 找名字第二个字是"三"的
SELECT * FROM users WHERE name LIKE '_三%';
```

- `%` → 匹配任意数量的字符（包括零个）
- `_` → 匹配一个字符

生活比喻：
- `'张%'` → 张三、张三丰、张大仙（以张开头的都行）
- `'%三%'` → 张三、王小三、赵三多（名字里有"三"的都行）
- `'_三%'` → 小三、王三、赵三丰（第二个字是"三"的）

### 排序（ORDER BY）

```sql
-- 按年龄从小到大排
SELECT * FROM users ORDER BY age ASC;

-- 按年龄从大到小排
SELECT * FROM users ORDER BY age DESC;
```

- `ORDER BY age` → 按age列排序
- `ASC` → 升序（从小到大），默认就是升序，不写也行
- `DESC` → 降序（从大到小）

**记忆方法：ASC = Ascending（上升），DESC = Descending（下降）。**

### 限制条数（LIMIT）

```sql
-- 只看前2条
SELECT * FROM users LIMIT 2;

-- 分页查询：跳过前2条，取接下来2条
SELECT * FROM users LIMIT 2, 2;
```

- `LIMIT 2` → 只取2条
- `LIMIT 2, 2` → 跳过2条，取2条（用于分页，第2页每页2条）

### 组合使用

```sql
SELECT name, age 
FROM users 
WHERE age > 20 
ORDER BY age DESC 
LIMIT 3;
```

翻译："从users表中，找出年龄大于20的，按年龄从大到小排序，只取前3条的name和age列。"

**SQL语句的执行顺序（不是书写顺序！）：**

```
1. FROM users        → 先确定从哪个表查
2. WHERE age > 20    → 筛选符合条件的行
3. SELECT name, age  → 从筛选结果中取指定的列
4. ORDER BY age DESC → 对结果排序
5. LIMIT 3           → 取前3条
```

## 4.3 更新数据（UPDATE）

```sql
UPDATE users SET age = 26 WHERE name = '张三';
```

翻译："把名字叫张三的记录，age改成26。"

- `UPDATE users` → 更新users表
- `SET age = 26` → 把age列设为26
- `WHERE name = '张三'` → 条件：名字是张三的

### 更新多个列

```sql
UPDATE users SET age = 26, email = 'new@163.com' WHERE name = '张三';
```

用逗号分隔多个要改的列。

### ⚠️ 超级警告

```sql
UPDATE users SET age = 26;
```

**如果不加WHERE条件，会把整张表所有记录的age都改成26！** 这在生产环境是灾难性的。

**永远记住：UPDATE一定要加WHERE条件！**

## 4.4 删除数据（DELETE）

```sql
DELETE FROM users WHERE name = '王五';
```

翻译："把名字叫王五的记录删掉。"

- `DELETE FROM users` → 从users表删除
- `WHERE name = '王五'` → 条件：名字是王五的

### ⚠️ 超级警告

```sql
DELETE FROM users;
```

**如果不加WHERE条件，会删除整张表的所有数据！** 跟UPDATE一样，DELETE也必须加WHERE。

### DELETE vs DROP的区别

| 操作 | 作用 | 区别 |
|------|------|------|
| DELETE FROM users | 删数据 | 只删数据，表结构还在，还能继续插入数据 |
| DROP TABLE users | 删表 | 表和数据一起删，表不存在了 |

```
DELETE → 把货架上的商品拿走，货架还在
DROP   → 连货架一起拆了
```

## 4.5 CRUD总结表

| 操作 | SQL关键字 | 翻译 |
|------|----------|------|
| 创建 | INSERT | "往表里放一条数据" |
| 读取 | SELECT | "从表里找数据" |
| 更新 | UPDATE | "改表里的数据" |
| 删除 | DELETE | "从表里删数据" |

**记住：UPDATE和DELETE一定要加WHERE！不然全表遭殃！**

---

# 第五章：用户权限管理 —— 谁能进仓库、能干什么

## 5.1 为什么需要用户权限管理

想象你的仓库有很多员工：
- 仓库管理员：什么都能干（进货、出货、改价格、盘点）
- 销售员：只能看商品信息，不能改
- 实习生：只能看部分信息

**MySQL的用户权限管理就是控制"谁能登录、能做什么操作"。**

## 5.2 MySQL用户长什么样

一个MySQL用户的完整身份是：`'用户名'@'主机名'`

```
'root'@'localhost'        → 只能从本机登录的root
'root'@'%'                → 能从任何机器登录的root
'repl'@'192.168.121.102'  → 只能从102这台机器登录的repl
'appuser'@'192.168.121.%' → 能从121网段任何机器登录的appuser
```

**关键理解：同样叫root，从不同IP连进来就是不同的用户，可以有不同的密码和权限。**

为什么需要 `'用户名'@'主机名'` 这种设计？

因为安全问题。比如：
- 管理员只允许在公司内网登录 → `'admin'@'192.168.168.%'`
- 应用服务器只允许从特定IP连接 → `'app'@'192.168.121.103'`
- 测试账号允许从任何地方登录 → `'test'@'%'`

`%` 是通配符，代表任意IP，跟SQL里LIKE的 `%` 一样的意思。

## 5.3 查看现有用户

```sql
SELECT user, host FROM mysql.user;
```

翻译："从mysql库的user表里，查看所有用户的用户名和主机名。"

MySQL自己有一个叫 `mysql` 的系统数据库，里面有一张 `user` 表，存着所有用户信息。

执行结果类似：

```
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| repl             | %         |
| admin            | %         |
+------------------+-----------+
```

前四个是MySQL自带的系统用户，别动它们。

## 5.4 创建用户

### 基本语法

```sql
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
```

逐词解释：
- `CREATE USER` → 创建用户
- `'zhangsan'@'localhost'` → 用户名zhangsan，只能从本机(localhost)登录
- `IDENTIFIED BY` → 密码是
- `'Test@123456'` → 密码内容

### 实际例子

```sql
-- 创建一个只能从本机登录的用户
CREATE USER 'zhangsan'@'localhost' IDENTIFIED BY 'Test@123456';

-- 创建一个能从任何机器登录的用户
CREATE USER 'lisi'@'%' IDENTIFIED BY 'Test@123456';

-- 创建一个只能从102这台机器登录的用户
CREATE USER 'wangwu'@'192.168.121.102' IDENTIFIED BY 'Test@123456';

-- 创建一个只能从121网段登录的用户
CREATE USER 'zhaoliu'@'192.168.121.%' IDENTIFIED BY 'Test@123456';
```

### MySQL 8.x 密码要求

MySQL 8.x 默认有密码策略，密码必须包含：
- 大写字母（A-Z）
- 小写字母（a-z）
- 数字（0-9）
- 特殊字符（如 @、#、$、! 等）

`Test@123456` 就满足要求：
- T → 大写字母 ✅
- est → 小写字母 ✅
- 123456 → 数字 ✅
- @ → 特殊字符 ✅

如果密码太简单（比如 `123456`），会报错：

```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

## 5.5 ⚠️ MySQL 8.x 重要变化

在MySQL 5.x时代，你可以用GRANT一条命令同时创建用户并授权：

```sql
-- MySQL 5.x 可以这样（自动创建用户+授权）
GRANT ALL ON *.* TO 'newuser'@'%' IDENTIFIED BY 'Test@123456';
```

**但MySQL 8.x不支持这样了！** 必须分两步：

```sql
-- 第一步：先创建用户
CREATE USER 'newuser'@'%' IDENTIFIED BY 'Test@123456';

-- 第二步：再授权
GRANT ALL ON *.* TO 'newuser'@'%';
```

如果你在MySQL 8.x用旧写法，会报错：

```
ERROR 1410 (42000): You are not allowed to create a user with GRANT
```

## 5.6 修改密码

```sql
-- 改自己的密码
ALTER USER USER() IDENTIFIED BY 'New@123456';

-- 改别人的密码（需要管理员权限）
ALTER USER 'zhangsan'@'localhost' IDENTIFIED BY 'New@123456';
```

- `ALTER USER` → 修改用户
- `USER()` → 当前登录的用户
- `IDENTIFIED BY` → 新密码是

## 5.7 删除用户

```sql
DROP USER 'zhangsan'@'localhost';
```

翻译："把zhangsan这个用户删掉。"

## 5.8 权限体系

MySQL的权限分三个层级，就像仓库的三级管理：

### 第一级：全局权限（所有库所有表）

```sql
GRANT ALL ON *.* TO 'admin'@'%';
```

- `*.*` → 第一个星号是所有数据库，第二个星号是所有表
- 翻译："给admin用户所有库所有表的全部权限。"

这是最高权限，相当于仓库的超级管理员。

### 第二级：库级权限（某个库的所有表）

```sql
GRANT ALL ON testdb.* TO 'dev'@'%';
```

- `testdb.*` → testdb库的所有表
- 翻译："给dev用户testdb库所有表的全部权限。"

相当于只管理某一个仓库的管理员。

### 第三级：表级权限（某个库的某张表）

```sql
GRANT SELECT ON testdb.users TO 'readonly'@'%';
```

- `testdb.users` → testdb库的users表
- 翻译："给readonly用户testdb库users表的查询权限。"

相当于只能看某个货架上东西的人。

## 5.9 常用权限一览

| 权限 | 说明 | 生活比喻 |
|------|------|---------|
| ALL | 所有权限 | 仓库管理员 |
| SELECT | 查询权限 | 只能看不能改 |
| INSERT | 插入权限 | 能往货架放东西 |
| UPDATE | 修改权限 | 能改货架上的东西 |
| DELETE | 删除权限 | 能从货架拿走东西 |
| CREATE | 建库建表权限 | 能造新货架 |
| DROP | 删库删表权限 | 能拆货架 |
| RELOAD | 刷新权限 | 能刷新系统设置 |
| REPLICATION SLAVE | 主从复制权限 | 能当从库 |
| REPLICATION CLIENT | 查看主从状态权限 | 能看主从状态 |

## 5.10 授权操作（GRANT）

### 基本语法

```sql
GRANT 权限 ON 库.表 TO '用户名'@'主机名';
```

### 实际例子

```sql
-- 给全部权限（管理员级别）
GRANT ALL ON *.* TO 'admin'@'%';

-- 给某个库的全部权限
GRANT ALL ON testdb.* TO 'dev'@'%';

-- 只给查询权限（只读用户）
GRANT SELECT ON testdb.* TO 'readonly'@'%';

-- 给查询和插入权限
GRANT SELECT, INSERT ON testdb.* TO 'dev2'@'%';

-- 给查询、插入、修改、删除权限（读写用户）
GRANT SELECT, INSERT, UPDATE, DELETE ON testdb.* TO 'app'@'%';
```

### WITH GRANT OPTION

```sql
GRANT ALL ON *.* TO 'superadmin'@'%' WITH GRANT OPTION;
```

`WITH GRANT OPTION` 的意思是：这个用户不仅能使用权限，还能把自己的权限授权给别人。

就像仓库管理员不仅能管理仓库，还能任命别人当管理员。

不加 `WITH GRANT OPTION` 的用户只能自己用权限，不能给别人授权。

## 5.11 查看用户权限

```sql
-- 查看自己的权限
SHOW GRANTS;

-- 查看某个用户的权限
SHOW GRANTS FOR 'zhangsan'@'localhost';
```

执行结果类似：

```
+--------------------------------------------------+
| Grants for zhangsan@localhost                    |
+--------------------------------------------------+
| GRANT SELECT ON testdb.* TO 'zhangsan'@'localhost'|
+--------------------------------------------------+
```

## 5.12 撤销权限（REVOKE）

### 基本语法

```sql
REVOKE 权限 ON 库.表 FROM '用户名'@'主机名';
```

注意：GRANT用TO，REVOKE用FROM。

### 实际例子

```sql
-- 撤销删除权限
REVOKE DELETE ON testdb.* FROM 'dev2'@'%';

-- 撤销全部权限
REVOKE ALL ON testdb.* FROM 'dev2'@'%';
```

## 5.13 刷新权限（FLUSH PRIVILEGES）

```sql
FLUSH PRIVILEGES;
```

翻译："让MySQL重新加载权限表，使刚才的修改立即生效。"

**什么时候需要执行？** 每次执行了以下操作后：
- CREATE USER
- ALTER USER
- GRANT
- REVOKE
- DROP USER
- SET PASSWORD

**养成习惯：改完权限就FLUSH PRIVILEGES。**

## 5.14 实际场景应用

| 场景 | 怎么配 | 理由 |
|------|--------|------|
| 给开发同事一个只读账号 | `GRANT SELECT ON testdb.* TO 'dev'@'192.168.121.%'` | 只能看，不能改，防止误操作 |
| 给应用一个读写账号 | `GRANT SELECT,INSERT,UPDATE,DELETE ON testdb.* TO 'app'@'192.168.121.%'` | 应用需要增删改查，但不能建表删表 |
| 给运维一个管理员账号 | `GRANT ALL ON *.* TO 'ops'@'192.168.121.%' WITH GRANT OPTION` | 运维需要全部权限 |
| 给主从复制一个账号 | `GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.121.%'` | 只需要复制权限 |
| 给数据分析一个只读账号 | `GRANT SELECT ON testdb.* TO 'analyst'@'%'` | 只查不改 |

## 5.15 用户权限管理总结

```
创建用户 → CREATE USER
删除用户 → DROP USER
改密码   → ALTER USER
授权     → GRANT ... TO
撤销权限 → REVOKE ... FROM
查看权限 → SHOW GRANTS FOR
刷新权限 → FLUSH PRIVILEGES
```

**记住MySQL 8.x铁律：先CREATE USER，再GRANT。不能一步到位。**

---

# 第六章：配置文件详解 —— MySQL的"说明书"

## 6.1 配置文件是什么

MySQL启动时会读一个配置文件，里面写着MySQL该怎么运行。

就像你新买了一个手机，第一次开机要设置语言、亮度、铃声……MySQL也一样，需要配置各种参数。

## 6.2 配置文件在哪

MySQL启动时会按以下顺序找配置文件：

```
/etc/my.cnf          ← 最常用的位置（系统级配置）
/etc/mysql/my.cnf    ← 备用位置
/usr/etc/my.cnf      ← 备用位置
~/.my.cnf            ← 用户级配置（当前用户的home目录）
```

找到哪个就用哪个，一般我们只改 `/etc/my.cnf`。

## 6.3 配置文件长什么样

```ini
[mysqld]
# 基础配置
port = 3306                          # MySQL监听的端口号
datadir = /var/lib/mysql             # 数据存放的目录
socket = /var/lib/mysql/mysql.sock   # socket文件路径（本地连接用）

# 字符集
character-set-server = utf8mb4       # 服务器默认字符集
collation-server = utf8mb4_general_ci  # 排序规则

# 连接数
max_connections = 200                # 最大同时连接数

# 日志
log-error = /var/log/mysqld.log      # 错误日志路径
slow_query_log = 1                    # 开启慢查询日志
long_query_time = 2                   # 超过2秒的SQL记录到慢日志

# 主从复制相关
log-bin = mysql-bin                  # 开启binlog（主从复制必须）
server-id = 1                        # 服务器唯一ID

# InnoDB引擎
innodb_buffer_pool_size = 256M       # InnoDB缓冲池大小

[client]
default-character-set = utf8mb4      # 客户端默认字符集
```

## 6.4 逐段详解

### [mysqld] 段

`[mysqld]` 是给MySQL服务端（mysqld进程）看的配置。**大部分配置都写在这里。**

### [client] 段

`[client]` 是给MySQL客户端（mysql命令行工具）看的配置。

### port = 3306

MySQL监听的端口号。3306是MySQL的默认端口，就像80是HTTP的默认端口一样。

如果一台机器上装了多个MySQL，可以改成不同的端口（3306、3307、3308...）。

### datadir = /var/lib/mysql

MySQL把所有数据库、表、数据存在这个目录下。

```
/var/lib/mysql/
  ├── mysql/           ← mysql系统库
  ├── testdb/          ← 你创建的testdb库
  │   ├── users.frm    ← users表的表结构
  │   ├── users.ibd    ← users表的数据
  │   └── ...
  ├── ibdata1          ← InnoDB系统数据
  ├── mysql.sock       ← socket文件
  └── auto.cnf         ← 自动配置
```

### socket = /var/lib/mysql/mysql.sock

socket文件是MySQL本地连接用的。

当你在本机执行 `mysql -u root -p` 时，MySQL客户端通过这个socket文件跟服务端通信，不走网络。

如果socket文件丢了或路径不对，会报错：

```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock'
```

### character-set-server = utf8mb4

MySQL默认字符集。设成utf8mb4就能存中文和emoji表情。

### collation-server = utf8mb4_general_ci

排序规则。`ci` 是 Case Insensitive（大小写不敏感）的缩写，意思是查询时不区分大小写：

```sql
-- 以下查询结果一样
SELECT * FROM users WHERE name = 'zhangsan';
SELECT * FROM users WHERE name = 'ZhangSan';
```

### max_connections = 200

最大同时连接数。表示最多允许多少个客户端同时连上来。

默认值通常是151，生产环境一般调大到500-1000。

如果连接数不够，新用户会报错 `Too many connections`。

### log-error = /var/log/mysqld.log

错误日志路径。MySQL启动失败、运行报错都会记录在这里。

**MySQL出问题时第一件事：看错误日志！**

```bash
cat /var/log/mysqld.log
# 或者只看最后50行
tail -50 /var/log/mysqld.log
```

### slow_query_log = 1

开启慢查询日志。1表示开启，0表示关闭。

### long_query_time = 2

慢查询阈值，单位是秒。超过这个时间的SQL语句会被记录到慢查询日志里。

设成2表示：执行时间超过2秒的SQL会被记录。

**慢查询日志是运维排查数据库性能问题的第一工具。**

比如用户反馈"网站很卡"，你就去看慢查询日志，找执行慢的SQL，然后优化它。

### log-bin = mysql-bin

开启binlog（二进制日志）。binlog记录了所有修改数据的操作（INSERT、UPDATE、DELETE）。

**主从复制必须有binlog**，因为从库就是通过读取主库的binlog来同步数据的。

### server-id = 1

MySQL服务器的唯一标识。**主从环境里每台MySQL的server-id必须不同！**

- 主库：server-id = 1
- 从库：server-id = 2

### innodb_buffer_pool_size = 256M

InnoDB缓冲池大小。这是MySQL最重要的性能参数。

InnoDB引擎会把热点数据（经常访问的数据）缓存在内存里，这个参数控制缓存多大。

**生产环境一般设为物理内存的60%-70%。** 比如服务器有8G内存，可以设：

```ini
innodb_buffer_pool_size = 5G
```

## 6.5 改完配置文件怎么办

**改完配置文件必须重启MySQL才能生效！**

```bash
systemctl restart mysqld
```

这跟Nginx不一样——Nginx可以reload不中断服务，但MySQL改配置文件必须重启。

## 6.6 查看当前配置

在MySQL里可以查看当前生效的配置参数：

```sql
-- 查看所有参数
SHOW VARIABLES;

-- 查看指定参数
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW VARIABLES LIKE 'long_query_time';
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'character_set%';
```

## 6.7 实操：开启慢查询日志

### 第1步：改配置文件

```bash
vi /etc/my.cnf
```

在 `[mysqld]` 段下加两行：

```ini
slow_query_log = 1
long_query_time = 2
```

### 第2步：重启MySQL

```bash
systemctl restart mysqld
```

### 第3步：验证

```sql
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW VARIABLES LIKE 'long_query_time';
```

看到 `slow_query_log = ON` 和 `long_query_time = 2.000000` 就说明生效了。

---

# 第七章：备份与恢复 —— 数据的"保险"

## 7.1 为什么要备份

想象你的仓库着火了，所有货物都烧了……

数据库也一样，可能因为以下原因丢数据：
- 硬盘坏了
- 有人误删了数据（DELETE FROM没加WHERE）
- 黑客攻击
- 程序bug导致数据错乱

**备份就是给数据上一份保险，出了事能恢复。**

## 7.2 mysqldump工具

MySQL自带了一个备份工具叫 `mysqldump`，它在命令行使用，不在MySQL里面用。

### 基本语法

```bash
mysqldump -u 用户名 -p [选项] 数据库名 [表名] > 备份文件.sql
```

逐部分解释：
- `mysqldump` → MySQL自带的备份工具
- `-u root` → 以root用户身份执行
- `-p` → 提示输入密码
- `数据库名` → 要备份哪个库
- `表名` → 可选，只备份某张表
- `>` → 重定向符号，把输出写到文件里
- `备份文件.sql` → 保存到哪个文件

**`>` 是Linux的重定向符号：把本来应该输出到屏幕的内容，改成写到文件里。**

## 7.3 备份操作

### 备份单个数据库

```bash
mysqldump -u root -p testdb > /tmp/testdb_backup.sql
```

翻译："用root用户备份testdb库，保存到/tmp/testdb_backup.sql文件。"

### 备份所有数据库

```bash
mysqldump -u root -p --all-databases > /tmp/all_backup.sql
```

`--all-databases` → 备份所有数据库。

### 备份单个表

```bash
mysqldump -u root -p testdb users > /tmp/users_backup.sql
```

翻译："备份testdb库里的users表。"

### 只备份表结构（不包含数据）

```bash
mysqldump -u root -p --no-data testdb > /tmp/testdb_struct.sql
```

`--no-data` → 不要数据，只要表结构（CREATE TABLE语句）。

### 备份多个指定数据库

```bash
mysqldump -u root -p --databases testdb1 testdb2 > /tmp/multi.sql
```

`--databases` → 后面跟多个库名，用空格分隔。

### 生产环境推荐写法

```bash
mysqldump -u root -p --single-transaction --master-data=2 testdb > /tmp/testdb.sql
```

- `--single-transaction` → 备份时不锁表，用事务保证一致性。**生产环境必加！**
- `--master-data=2` → 在备份文件里记录binlog位置，方便做主从同步。

## 7.4 备份文件长什么样

打开看看就知道了：

```bash
cat /tmp/testdb_backup.sql
```

里面其实就是SQL语句：

```sql
-- MySQL dump 10.13  Distrib 8.1.0, for Linux (x86_64)
--
-- Host: localhost    Database: testdb
--

--
-- Table structure for table users
--

DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `age` int DEFAULT NULL,
  `email` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

--
-- Dumping data for table users
--

LOCK TABLES `users` WRITE;
INSERT INTO `users` VALUES (1,'张三',25,'zhangsan@163.com'),(2,'李四',30,'lisi@163.com'),(3,'王五',28,'wangwu@163.com');
UNLOCK TABLES;
```

**mysqldump的本质：把数据库结构和数据导出成SQL语句文本文件。** 恢复时就是重新执行一遍这些SQL。

## 7.5 恢复操作

### 基本语法

```bash
mysql -u 用户名 -p 数据库名 < 备份文件.sql
```

注意：恢复用的是 `mysql` 命令，不是 `mysqldump`。

- `<` → 重定向符号，把文件内容读进去（跟备份的 `>` 方向相反）

### 恢复单个数据库

```bash
mysql -u root -p testdb < /tmp/testdb_backup.sql
```

翻译："用root用户，把/tmp/testdb_backup.sql文件里的SQL语句在testdb库中执行。"

**前提：testdb库必须已经存在。** 如果不存在，先创建：

```bash
mysql -u root -p -e "CREATE DATABASE testdb"
```

`-e` → 执行后面的SQL语句然后退出。

### 恢复所有数据库

```bash
mysql -u root -p < /tmp/all_backup.sql
```

不需要指定数据库名，因为备份文件里包含了CREATE DATABASE语句。

## 7.6 > 和 < 的含义

| 符号 | 名称 | 方向 | 用途 |
|------|------|------|------|
| `>` | 输出重定向 | 往外吐 | 备份：把数据导出到文件 |
| `<` | 输入重定向 | 往里喂 | 恢复：把文件数据读进数据库 |

**记忆方法：**
- `>` → 像一个箭头把数据射到文件里 → **备份**
- `<` → 像一个漏斗把文件灌进数据库 → **恢复**

## 7.7 实操：完整备份恢复流程

### 第1步：建库建表插数据

```sql
CREATE DATABASE backup_test;
USE backup_test;
CREATE TABLE test_table (id INT, name VARCHAR(50));
INSERT INTO test_table VALUES (1, 'test1'), (2, 'test2');
```

### 第2步：备份

```bash
mysqldump -u root -p backup_test > /tmp/backup_test.sql
```

### 第3步：模拟故障（删库）

```sql
DROP DATABASE backup_test;
```

### 第4步：恢复

```bash
# 先建库
mysql -u root -p -e "CREATE DATABASE backup_test"

# 再恢复数据
mysql -u root -p backup_test < /tmp/backup_test.sql
```

### 第5步：验证

```sql
USE backup_test;
SELECT * FROM test_table;
```

看到数据回来了就说明恢复成功。

## 7.8 备份恢复总结

| 操作 | 命令 | 符号 |
|------|------|------|
| 备份 | `mysqldump -u root -p 库名 > 文件` | `>` |
| 恢复 | `mysql -u root -p 库名 < 文件` | `<` |

**备份往外吐 `>`，恢复往里喂 `<`。**

## 7.9 定时备份（了解）

生产环境不能手动备份，要用定时任务自动备份。

```bash
# 编辑定时任务
crontab -e

# 每天凌晨2点自动备份
0 2 * * * mysqldump -u root -pYourPassword --all-databases > /backup/mysql_$(date +\%Y\%m\%d).sql
```

- `0 2 * * *` → 每天凌晨2点
- `$(date +%Y%m%d)` → 日期变量，比如20260628，文件名就是 mysql_20260628.sql

---

# 第八章：主从复制 —— 数据的"分身术"

## 8.1 什么是主从复制

想象你有两个仓库，一个在北京（主库），一个在上海（从库）。

每次北京仓库进了新货，上海仓库自动同步进货信息。

**主从复制就是：主库的数据变更自动同步到从库。**

```
主库（Master）→ 写入数据 → 记录到binlog
                          ↓
从库（Slave）→ 读取binlog → 重放SQL → 数据同步
```

## 8.2 为什么要主从复制

| 好处 | 说明 |
|------|------|
| 读写分离 | 主库负责写，从库负责读，减轻压力 |
| 数据备份 | 从库就是一份实时备份 |
| 高可用 | 主库挂了，从库可以顶上 |
| 扩展性 | 读压力大时可以加更多从库 |

生活比喻：
- 主库 = 总部的账本（记账在这里）
- 从库 = 分公司的账本副本（自动同步，用来查账）

## 8.3 主从复制的原理

主从复制分三步：

### 第1步：主库记录binlog

主库每次执行INSERT/UPDATE/DELETE，都会把操作记录到binlog（二进制日志）里。

```
主库执行：INSERT INTO users VALUES (4, '赵六')
  ↓
binlog记录：这条INSERT语句
```

### 第2步：从库IO线程拉取binlog

从库有一个IO线程，连接到主库，把主库的binlog拉过来，存到自己的relaylog（中继日志）里。

```
从库IO线程 → 连接主库 → 读取binlog → 存到relaylog
```

### 第3步：从库SQL线程重放relaylog

从库还有一个SQL线程，读取relaylog里的SQL语句，在从库上执行一遍。

```
从库SQL线程 → 读取relaylog → 执行INSERT语句 → 从库也有了这条数据
```

### 完整流程图

```
主库                          从库
┌──────────────┐           ┌──────────────────┐
│  写入数据     │           │                  │
│      ↓       │           │  IO线程           │
│  记录binlog  │ ────────→ │  读取binlog       │
│              │           │      ↓            │
│              │           │  存到relaylog     │
│              │           │      ↓            │
│              │           │  SQL线程重放      │
│              │           │      ↓            │
│              │           │  从库也有数据了    │
└──────────────┘           └──────────────────┘
```

### 两个线程

从库有两个线程在工作：

| 线程 | 作用 | 状态字段 |
|------|------|---------|
| IO线程 | 从主库拉取binlog | Slave_IO_Running |
| SQL线程 | 在从库重放SQL | Slave_SQL_Running |

**两个线程都必须是Yes才算正常！**

## 8.4 主从复制配置教程

### 环境准备

- 主库：hadoop101（192.168.121.101）
- 从库：hadoop102（192.168.121.102）
- 两台都装好MySQL，能正常登录

### 第1步：配置主库（101）

#### 修改主库配置文件

```bash
vi /etc/my.cnf
```

在 `[mysqld]` 段下加两行：

```ini
log-bin=mysql-bin       # 开启binlog
server-id=1             # 主库ID设为1
```

- `log-bin=mysql-bin` → 开启二进制日志，文件名以mysql-bin开头
- `server-id=1` → 服务器唯一标识，主从环境里每台必须不同

重启MySQL使配置生效：

```bash
systemctl restart mysqld
```

#### 登录主库创建复制用户

```bash
mysql -u root -p
```

```sql
-- 创建复制专用用户
CREATE USER 'repl'@'%' IDENTIFIED BY 'Test@123456';

-- 给复制权限
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- 刷新权限
FLUSH PRIVILEGES;
```

为什么需要单独的复制用户？因为从库需要用这个用户连接主库来读取binlog。

- `REPLICATION SLAVE` → 这个权限允许用户读取主库的binlog
- `'repl'@'%'` → 允许从任何IP连接（生产环境应该限制IP）

#### 查看主库状态

```sql
SHOW MASTER STATUS;
```

执行结果：

```
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     874  |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

**记下 File 和 Position 的值！** 后面配置从库时要用。

- `File: mysql-bin.000001` → 当前binlog文件名
- `Position: 874` → 当前binlog的位置（从库要从这个位置开始读取）

### 第2步：配置从库（102）

#### 修改从库配置文件

```bash
vi /etc/my.cnf
```

在 `[mysqld]` 段下加一行：

```ini
server-id=2             # 从库ID设为2（不能跟主库一样）
```

从库不需要开log-bin（除非它同时也是别人的主库）。

重启MySQL：

```bash
systemctl restart mysqld
```

#### 登录从库配置主库信息

```bash
mysql -u root -p
```

```sql
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='192.168.121.101',
    SOURCE_USER='repl',
    SOURCE_PASSWORD='Test@123456',
    SOURCE_LOG_FILE='mysql-bin.000001',
    SOURCE_LOG_POS=874;
```

逐行解释：
- `CHANGE REPLICATION SOURCE TO` → "我要配置主库信息了"（MySQL 8.x语法）
- `SOURCE_HOST='192.168.121.101'` → 主库的IP地址
- `SOURCE_USER='repl'` → 用repl用户连接主库
- `SOURCE_PASSWORD='Test@123456'` → repl用户的密码
- `SOURCE_LOG_FILE='mysql-bin.000001'` → 从主库的哪个binlog文件开始读（就是刚才SHOW MASTER STATUS看到的File）
- `SOURCE_LOG_POS=874` → 从binlog的哪个位置开始读（就是刚才SHOW MASTER STATUS看到的Position）

**注意：MySQL 8.0之前用 `MASTER_HOST`、`MASTER_USER` 等关键字，8.0之后改成了 `SOURCE_HOST`、`SOURCE_USER`。功能完全一样，只是名字变了。**

旧语法（MySQL 5.x）：

```sql
CHANGE MASTER TO
    MASTER_HOST='192.168.121.101',
    MASTER_USER='repl',
    MASTER_PASSWORD='Test@123456',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=874;
```

#### 启动复制

```sql
START SLAVE;
```

翻译："开始主从复制。"

**注意：MySQL 8.0.22+ 也支持 `START REPLICA;` 语法，跟 `START SLAVE;` 完全等价。**

#### 查看复制状态

```sql
SHOW SLAVE STATUS\G
```

`\G` 不是分号结尾，而是以 `\G` 结尾。它的作用是把输出竖着显示，方便阅读。

**重点看这两行：**

```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

**两个都是Yes才说明主从复制正常！**

### 各种状态含义

| 字段 | 值 | 含义 |
|------|-----|------|
| Slave_IO_Running | Yes | IO线程正常运行，正在从主库拉取binlog |
| Slave_IO_Running | Connecting | IO线程正在连接主库（还没连上） |
| Slave_IO_Running | No | IO线程没运行（出问题了） |
| Slave_SQL_Running | Yes | SQL线程正常运行，正在重放relaylog |
| Slave_SQL_Running | No | SQL线程没运行（出问题了） |

### 常见问题排查

#### 问题1：Slave_IO_Running: Connecting

说明从库连不上主库。可能原因：

1. **密码输错了** → 重新检查SOURCE_PASSWORD
2. **主库防火墙没开3306端口** → 在主库执行 `firewall-cmd --add-port=3306/tcp --permanent && firewall-cmd --reload`，或者直接 `systemctl stop firewalld`
3. **repl用户权限不对** → 在主库检查 `SHOW GRANTS FOR 'repl'@'%';`
4. **网络不通** → 在从库 `ping 192.168.121.101` 测试

排查方法：在从库用命令行测试连接主库：

```bash
mysql -u repl -p -h 192.168.121.101
```

- 如果报 `Access denied` → 密码错或用户权限问题
- 如果报 `Can't connect` → 网络或防火墙问题
- 如果能连上 → 说明是CHANGE REPLICATION SOURCE配置问题

#### 问题2：Slave_SQL_Running: No

说明SQL线程执行报错了。查看 `Last_Error` 字段看具体错误。

常见的解决方法：

```sql
-- 跳过一条错误的SQL（谨慎使用）
STOP SLAVE;
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE;
```

### 第3步：验证主从同步

#### 在主库（101）上写数据

```sql
CREATE DATABASE repl_test;
USE repl_test;
CREATE TABLE test (id INT, name VARCHAR(50));
INSERT INTO test VALUES (1, 'hello master-slave');
```

#### 在从库（102）上读数据

```sql
SHOW DATABASES;
-- 应该能看到 repl_test

USE repl_test;
SELECT * FROM test;
-- 应该能看到 (1, 'hello master-slave')
```

数据同步过来了，说明主从复制成功！

### 第4步：测试故障切换

#### 停止从库复制

```sql
STOP SLAVE;
```

#### 在主库插入新数据

```sql
INSERT INTO repl_test.test VALUES (2, 'new data');
```

#### 从库查不到（因为复制停了）

```sql
SELECT * FROM repl_test.test;
-- 只有(1, 'hello master-slave')，没有(2, 'new data')
```

#### 重新启动复制

```sql
START SLAVE;
```

#### 再查，数据同步过来了

```sql
SELECT * FROM repl_test.test;
-- 现在有两条：(1, 'hello master-slave') 和 (2, 'new data')
```

## 8.5 主从复制常用命令总结

| 操作 | 命令 |
|------|------|
| 启动复制 | `START SLAVE;` |
| 停止复制 | `STOP SLAVE;` |
| 查看状态 | `SHOW SLAVE STATUS\G` |
| 重置从库配置 | `RESET SLAVE;` |
| 查看主库状态 | `SHOW MASTER STATUS;` |
| 查看binlog事件 | `SHOW BINLOG EVENTS;` |

## 8.6 主从复制的注意事项

### 从库默认只读

从库通常设为只读模式，防止有人在从库上写数据导致数据不一致：

```sql
SET GLOBAL super_read_only = ON;
```

**从库只能读不能写，写操作只能在主库做。**

### 主从延迟

主库写入后，从库同步有一点点延迟（通常是毫秒级），因为：
1. IO线程要从主库拉取binlog（网络传输需要时间）
2. SQL线程要在从库重放SQL（执行需要时间）

如果从库压力大（SQL执行慢），延迟会增大。

查看延迟：

```sql
SHOW SLAVE STATUS\G
-- 看 Seconds_Behind_Master 字段
-- 0 = 没有延迟
-- 100 = 落后主库100秒
```

### 主库挂了怎么办

主库故障后，需要手动把从库提升为主库（或者用MHA等工具自动切换）：

```sql
-- 在从库上执行
STOP SLAVE;
RESET SLAVE ALL;
SET GLOBAL read_only = OFF;
```

这时候从库就变成了一个独立的主库，可以读写了。

## 8.7 主从复制面试标准答案

> MySQL主从复制基于binlog实现。主库开启binlog记录所有写操作，从库的IO线程连接主库读取binlog存到本地relaylog，SQL线程读取relaylog重放SQL实现数据同步。配置时主库需要创建REPLICATION SLAVE权限的复制用户，从库用CHANGE REPLICATION SOURCE指定主库地址、用户、binlog文件名和位置。通过SHOW SLAVE STATUS查看Slave_IO_Running和Slave_SQL_Running是否都是Yes。主从复制用于读写分离、数据备份和高可用。

---

# 第九章：MySQL重置root密码

## 9.1 什么时候需要重置密码

忘了root密码，登不进MySQL了。

## 9.2 重置步骤（MySQL 8.x）

```bash
# 1. 停掉MySQL
systemctl stop mysqld

# 2. 以mysql用户身份跳过权限验证启动
sudo -u mysql mysqld --skip-grant-tables &

# 3. 直接登录（不需要密码）
mysql -u root

# 4. 刷新权限并改密码
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'New@123456';
exit

# 5. 杀掉mysqld进程
killall mysqld

# 6. 正常启动
systemctl start mysqld

# 7. 用新密码登录
mysql -u root -p
```

### 为什么要用 sudo -u mysql

MySQL 8.x 不允许以root用户身份运行mysqld，会报错：

```
Fatal error: Please read "Security" section of the manual to find out how to run mysqld as root!
```

所以要用 `sudo -u mysql` 切换到mysql用户来运行。

### --skip-grant-tables 是什么意思

这个参数让MySQL启动时跳过权限验证，任何人不用密码就能登录。**只能用于密码重置，绝对不能在生产环境长期使用！**

---

# 第十章：全章总结

## MySQL运维核心知识地图

```
MySQL运维
├── 基础操作
│   ├── 库操作：CREATE / USE / DROP
│   ├── 表操作：CREATE / DESC / ALTER / DROP
│   └── 数据操作：INSERT / SELECT / UPDATE / DELETE
│
├── 用户权限
│   ├── 创建用户：CREATE USER
│   ├── 授权：GRANT
│   ├── 撤销：REVOKE
│   └── 刷新：FLUSH PRIVILEGES
│
├── 配置文件（/etc/my.cnf）
│   ├── 基础配置：port / datadir / socket
│   ├── 字符集：character-set-server = utf8mb4
│   ├── 日志：log-error / slow_query_log
│   └── 性能：innodb_buffer_pool_size / max_connections
│
├── 备份恢复
│   ├── 备份：mysqldump ... > file.sql
│   ├── 恢复：mysql ... < file.sql
│   └── 定时备份：crontab + mysqldump
│
└── 主从复制
    ├── 原理：binlog → IO线程 → relaylog → SQL线程
    ├── 主库配置：log-bin + server-id + repl用户
    ├── 从库配置：server-id + CHANGE REPLICATION SOURCE
    ├── 验证：SHOW SLAVE STATUS（两个Yes）
    └── 故障排查：IO线程 / SQL线程
```

## 面试高频考点

| 考点 | 一句话回答 |
|------|-----------|
| CRUD是什么 | Create(INSERT)、Read(SELECT)、Update(UPDATE)、Delete(DELETE) |
| 主键是什么 | 唯一标识一条记录的列，不能重复不能为空 |
| utf8和utf8mb4区别 | utf8最多3字节存不了emoji，utf8mb4最多4字节全能存 |
| MySQL 8.x创建用户 | 必须先CREATE USER再GRANT，不能一步到位 |
| 备份用什么命令 | mysqldump，备份用>重定向，恢复用<重定向 |
| 慢查询日志 | 记录执行时间超过阈值的SQL，排查性能问题第一工具 |
| 主从复制原理 | 主库binlog → 从库IO线程拉取存relaylog → SQL线程重放 |
| 主从怎么看状态 | SHOW SLAVE STATUS，看Slave_IO_Running和Slave_SQL_Running是否Yes |
| innodb_buffer_pool_size | InnoDB缓存池，生产环境设为物理内存60-70% |
| UPDATE/DELETE注意什么 | 必须加WHERE条件，否则全表遭殃 |

---

> **这篇文档涵盖了MySQL运维的所有核心内容。每一章都可以独立翻阅，忘了哪个知识点就翻到对应章节复习。祝你学习顺利！**
