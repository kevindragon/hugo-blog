---
title: "MySQL导入csv数据"
date: 2022-12-01T15:46:07+08:00
tags: [MySQL, 数据库]
categories: [数据库]
---

# MySQL导入csv数据

在大数据时代，我们每天面临着大量的数据需要处理。表格型数据是最常见的一种，特别是财务从业者。如何快速的把CSV类型的数据导入到数据库进行快速处理呢？今天我们就来介绍其中一种。

<!-- more -->

## LOAD DATA语句定义

我们先来看看MySQL的 `LOAD DATA` 语句的定义

```sql
LOAD DATA
    [LOW_PRIORITY | CONCURRENT] [LOCAL]
    INFILE 'file_name'
    [REPLACE | IGNORE]
    INTO TABLE tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [CHARACTER SET charset_name]
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [IGNORE number {LINES | ROWS}]
    [(col_name_or_user_var
        [, col_name_or_user_var] ...)]
    [SET col_name={expr | DEFAULT}
        [, col_name={expr | DEFAULT}] ...]
```

`LOAD DATA` 语句以非常高的速度将文本文件中的行读入表中。 该文件可以从服务器主机或客户端主机读取，具体取决于是否给出了 LOCAL 修饰符。 LOCAL 还会影响数据解释和错误处理。

LOAD DATA 是 SELECT ... INTO OUTFILE 的补充。要将表中的数据写入文件，请使用 SELECT ... INTO OUTFILE。 要将文件读回表中，请使用 LOAD DATA。 两个语句的 FIELDS 和 LINES 子句的语法相同。

- **LOW_PRIORITY**

    使用 LOW_PRIORITY 修饰符，LOAD DATA 语句的执行被延迟，直到没有其他客户端正在从表中读取。 这只会影响仅使用表级锁定的存储引擎（例如 MyISAM、MEMORY 和 MERGE）。

- **CONCURRENT**

    使用 CONCURRENT 修饰符和满足并发插入条件的 MyISAM 表（即中间不包含空闲块），其他线程可以在执行 LOAD DATA 时从表中检索数据。 这个修饰符会稍微影响 LOAD DATA 的性能，即使没有其他线程同时使用该表。

- **INFILE**

    INFILE 后面接文件路径，可以使用绝对路径和相对路径。其中相对路径是你使用mysql登录mysql server时所在的目录

- **INTO TABLE**

    INTO TABLE 没什么好说的，后面指定的是表的名称

- **FIELDS TERMINATED BY, LINES TERMINATED BY**

    FIELDS TERMINATED BY ‘string’

    这两个修饰符用法类型，一个是指定字段之间使用什么样的字符串 `string` 分割，一个是每一行之间使用什么样的字符串 `string` 分割。 `string` 可以超过一个字符。

- **IGNORE {number} LINES**

    指定忽略的行数，从第一行算。一般像csv这样的文件，第一行一般是一个表头，表示第一列的名称。{number}换成1表示忽略第一行。

- **SET**

    SET 子句能够在将结果分配给列之前对其值执行预处理转换。


对于 `SET` 接下来在导入数据的时候还会详细介绍。

## 导入CSV数据

```sql
LOAD DATA INFILE 'data.csv'
INTO TABLE TABLE_NAME
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(id, name, value);
```

如果csv文件的每一个行的所有列都有值，则可以顺利的导入，可以看到输出内容

```
Query OK, 407 rows affected (0.02 sec)
Records: 407  Deleted: 0  Skipped: 0  Warnings: 0
```

但是如果csv里面的数据类型不匹配；或者某个值为空，而schema又不允许为空；又或者某一列的值你在导入的时候是不需要的。这时候需要使用到输入预处理过程了，在MySQL的 `LOAD DATA` 叫做 ****`Input Preprocessing`****

### 输入预处理

LOAD DATA 语法中的每个 col_name_or_user_var 实例都是列名或用户变量。 对于用户变量，SET 子句使您能够在将结果分配给列之前对其值执行预处理转换。

SET 子句中的用户变量可以多种方式使用。 下面的例子直接将第一个输入列作为t1.column1的值，将第二个输入列赋值给一个用户变量，该用户变量经过除法运算后用于t1.column2的值：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, @var1)
  SET column2 = @var1/100;
```

这里的 `@var1` 保存的是每一行第二列的值，而数据库表的第二个字段为 column2，这样才可以插入到column2字段当中。

SET 子句可用于提供不是从输入文件派生的值。 以下语句将 column3 设置为当前日期和时间：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, column2)
  SET column3 = CURRENT_TIMESTAMP;
```

您还可以通过将输入值分配给用户变量而不将该变量分配给任何表列来丢弃输入值：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, @dummy, column2, @dummy, column3);
```

当mysql的表当中某个字段不允许为空，默认值为 `0` ，而csv文件当中恰好有空值的时候，可以像下面这样处理：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, @vColumn2, column3)
  SET column2 = IFNULL(0, @vColumn2);
```

这样就可以处理第二列为空的时候，默认插入值 `0` 了

列/变量列表和 SET 子句的使用受以下限制：

- SET 子句中的赋值应该只有赋值运算符左侧的列名。
- 您可以在 SET 赋值的右侧使用子查询。 返回要分配给列的值的子查询可能只是标量子查询。 此外，您不能使用子查询从正在加载的表中进行选择。
- 不会为列/变量列表或 SET 子句处理被 IGNORE number LINES 子句忽略的行。
- 加载固定行格式的数据时不能使用用户变量，因为用户变量没有显示宽度。

### 转义

需要被导入的数据需要特别注意转义，否则就会出现下面的错误

```
Row 4322 doesn't contain data for all columns
```

比如说 `"\"` 这样的原始字符串，在csv文件当中应该要写成 `"\\"` ，注意是两个反斜杠。

## 权限

按上面的 `LOAD DATA` 语句导入数据，很可能会碰到下面的情况。

```sql
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

查看系统变量 `secure-file-priv`

```sql
mysql> SHOW VARIABLES LIKE "secure_file_priv";

+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv | NULL  |
+------------------+-------+
```

可以看到 `secure_file_priv` 值为 `NULL`

### `secure_file_priv` 配置项

| Command-Line Format | --secure-file-priv=dir_name |
| --- | --- |
| System Variable | secure_file_priv |
| Scope | Global |
| Dynamic | No |
| Type | String |
| Default Value | platform specific |
| Valid Values | empty string
dirname
NULL |

该变量用于限制数据导入和导出操作，例如由 `LOAD DATA` 和 `SELECT ... INTO OUTFILE` 语句以及 `LOAD_FILE()` 函数执行的操作权限。 这些操作仅允许具有 `FILE` 权限的用户使用。

secure_file_priv 可以设置为如下值：

- 空值，不做目录限制，即任何目录均可以；
- 指定目录，MySQL 会限制只能从该目录导入、或导出到该目录。目录必须已存在，MySQL 不会自动创建该目录；
- `NULL` ， 服务器禁用导入和导出操作。

默认值是特定于平台的，取决于 `INSTALL_LAYOUT` CMake 选项的值，如下表所示。 如果您从源代码构建，要明确指定默认的 `secure_file_priv` 值，请使用 `INSTALL_SECURE_FILE_PRIVDIR` CMake 选项。

| INSTALL_LAYOUT Value | Default secure_file_priv Value |
| --- | --- |
| STANDALONE, WIN | NULL (>= MySQL 5.7.16), empty (< MySQL 5.7.16) |
| DEB, RPM, SLES, SVR4 | /var/lib/mysql-files |
| Otherwise | mysql-files under the CMAKE_INSTALL_PREFIX value |

要为 libmysqld 嵌入式服务器设置默认的 secure_file_priv 值，请使用 `INSTALL_SECURE_FILE_PRIV_EMBEDDEDDIR` CMake 选项。 此选项的默认值为 NULL。

服务器在启动时检查 `secure_file_priv` 的值，如果该值不安全，则将警告写入错误日志。 如果非 NULL 值为空，或者该值为数据目录或其子目录，或者所有用户都可以访问的目录，则认为它是不安全的。 如果 `secure_file_priv` 设置为不存在的路径，则服务器会将错误消息写入错误日志并退出。

## 配置

如果要取消这种限制，可以修改 `my.cnf` 配置文件，在 `[mysqld]` 项加入下面的配置

```
secure_file_priv=
```

值留为空，表示不做任何限制。