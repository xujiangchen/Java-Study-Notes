- [特殊属性字段](#特殊属性字段)
  - [1、bit](#1bit)
  - [2、blob](#2blob)
  - [3、binary](#3binary)

## 特殊属性字段
### 1、bit
可以使用b'value'符号写位字段值。value是一个用0和1写成的二进制值。

位字段符号可以方便指定分配给BIT列的值：

```sql
mysql> CREATE TABLE t (b BIT(8));

mysql> INSERT INTO t SET b = b'11111111';

mysql> INSERT INTO t SET b = b'1010';
```

**查询**
```sql
-- 十进制显示
select field+0 from tablename; 
-- 二进制
select bin(field+0) from tablename;
-- 八进制
select oct(field+0) from tablename; 
-- 十六进制
select hex(field+0) from tablename; 
```

### 2、blob

在MySQL中Blob是一个二进制的对象，它是一个可以存储大量数据的容器(如图片，音乐等等)，且能容纳不同大小的数据，在MySQL中有四种Blob类型，他们的区别就是可以容纳的信息量不容分别是以下四种:

- TinyBlob类型  最大能容纳255B的数据
- Blob类型  最大能容纳65KB的
- MediumBlob类型  最大能容纳16MB的数据
- LongBlob类型  最大能容纳4GB的数据

```sql
-- 常用类型为 utf8mb4，gbk，utf8
select CONVERT(`field` USING utf8mb4)AS dataConvertValue from tableName;
```

### 3、binary

binary 和 varbinary 类型类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字节字符串
- binary:固定长度二进制字符串,**指定长度后，不足最大长度的，将在它们右边填充 “\0” 补齐，以达到指定长度**
- varbinary:可变长度二进制字符串

```sql
-- 常用类型为 utf8mb4，gbk，utf8
select CONVERT(`field` USING utf8mb4)AS dataConvertValue from tableName;
```
