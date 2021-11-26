- [特殊属性字段](#特殊属性字段)
  - [1、Blob和Clob](#1blob和clob)

## 特殊属性字段
### 1、Blob和Clob
BLOB和CLOB都是大字段类型，BLOB是按二进制来存储的，而CLOB是可以直接存储文字的。其实两个是可以互换的的，或者可以直接用LOB字段代替这两个。但是为了更好的管理ORACLE数据库，通常像图片、文件、音乐等信息就用BLOB字段来存储，先将文件转为二进制再存储进去。而像文章或者是较长的文字，就用CLOB存储，这样对以后的查询更新存储等操作都提供很大的方便。  
```sql
-- dbms_lob.substr()函数用来操作的大型对象，叫做大型对象定位器,但是此方法查询出的数据长度不可超过4000
-- 查询Blob和Clob属性字段，以文本格式显示
select utl_raw.cast_to_varchar2("field") from table

select utl_raw.cast_to_varchar2(dbms_lob.substr("field")) from table

-- dbms_lob.instr(源字符串, 目标字符串, 起始位置, 匹配序号)

-- 以Blob属性字段为条件查询
select * from table t where dbms_lob.instr(blob_msg,utl_raw.CAST_TO_RAW('同意'),1,1)>0;

-- 以Clob属性字段为条件查询
select * from table t where dbms_lob.instr(msg_clob,'liaoyuhan',1,1) > 0;
```