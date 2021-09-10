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