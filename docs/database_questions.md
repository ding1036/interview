<a id = "jump">[首页](/README.md)</a>

```sql
SELECT * FROM student
SELECT * FROM course
SELECT * FROM sc
```

1. 使用标准 SQL 嵌套语句查询选修课程名称为’effect c++’ 的学员学号和姓名

```sql
SELECT sname,sno FROM student
WHERE sno IN(
SELECT sno FROM sc WHERE cno=(
SELECT cno FROM course WHERE cname='effect c++'))
```

2.使用标准 SQL 嵌套语句查询选修课程编号为’002’ 的学员姓名和年龄

```sql
SELECT sname,sage FROM student
WHERE sno IN(
SELECT sno FROM sc
WHERE sc.cno='002')
```

3. 使用标准 SQL 嵌套语句查询不选修课程编号为’002’ 的学员姓名和年龄

```sql
SELECT sname,sage FROM student
WHERE sno not in (
SELECT sno FROM sc WHERE cno='002')
```

4. 使用标准 SQL 嵌套语句查询选修全部课程的学员姓名和年龄

```sql
SELECT sname,sage FROM student
WHERE sno IN(
SELECT sc.sno FROM sc RIGHT JOIN course c ON sc.cno=c.cno
GROUP BY sno HAVING COUNT(*)=(select DISTINCT count(*) from course))
--or
--GROUP BY sno HAVING COUNT(*)=(select count(DISTINCT cname) from course))
```

5.查询选修了课程的学员人数

```sql
SELECT count(distinct sno) FROM sc
```

6. 查询选修课程超过 2 门的学员学号和年龄

```sql
SELECT sname,sage FROM student
WHERE sno in(
SELECT sno FROM sc GROUP BY sno HAVING count(cno)>2)
```

7. 找出没有选修过“李明” 老师讲授课程的所有学生姓名

```sql
SELECT sno,sname FROM student WHERE sno not in
(SELECT sno FROM sc WHERE cno =(
SELECT cno FROM course WHERE cteacher='李明'))
```

[toTop](#jump)