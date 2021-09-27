<a id = "jump">[首页](/README.md)</a>


# idea提示" cannot access xxx.class"的解决方法


解决思路：

1、重启IDEA

2、重启无效果，File -> Invalidate Caches -> Invalidate and Restart


[toTop](#jump)

# DB2 
## db2 合并字符

```sql
SELECT [分组的字段],LISTAGG([需要聚合的字段名], ',') WITHIN GROUP(ORDER BY [排序的字段名]) AS employees
FROM [表名] GROUP BY [分组的字段名] ;                  --注意：需要DB2 9.7以后的版本才支持
```

[toTop](#jump)


# datax
## Datax执行命令后出现中文乱码的解决办法
控制台出现乱码：直接输入``CHCP 65001``回车

## write连接db2数据库失败 

ERROR RetryUtil - Exception when calling callable,异常Msg:Code:[DBUtilErrorCode-10], Description:[连接数据库失败. 请检查您的 账号、密码、数据库名称、IP、Port或者向 DBA 寻求帮助(注意网络环境).]. - 具体错误信息为：java.sql.SQLException: No suitable driver found for [“jdbc:mysql://localhost:3306/datax”]?yearIsDateType=false&zeroDateTimeBehavior=convertToNull&tinyInt1isBit=false&rewriteBatchedStatements=true
解决方法：

首先检查数据库的连接地址、账号、密码等，如果都正确则参考以下方法

因为json配置文件中的writer中的 jdbcUrl 不能使用[]中括号括起来，即表示只能有一个写入的库。
