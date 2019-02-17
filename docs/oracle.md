<a id = "jump">[首页](/README.md)</a>



# oracle存储函数和函数的区别

定义关键字不同，存储函数是``procedure`` , 函数是 ``function``

存储过程不需要声明返回类型，可以返回多个值
函数需要声明返回类型

# oracle触发器类型和出发事件
类型 ：before , after ,instead of
事件 ：insert ,delete ,update

# 触发器启用或者禁用

启用：Alter trigger trigger_name enable
禁用：Alter trigger trigger_name disable
禁用所有触发器: Alter table table_name disable all triggers
启用所有触发器: Alter table table_name enable all triggers
重新编译触发器：Alter trigger trigger_name compile


[toTop](#jump)