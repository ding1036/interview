<a id = "jump">[首页](/README.md)</a>


# 用例图中三种关系

## 包含(include)

包含关系：使用包含（Inclusion）用例来封装一组跨越多个用例的相似动作（行为片断），以便多个基（Base）用例复用。基用例控制与包含用例的 关系，以及被包含用例的事件流是否会插入到基用例的事件流中。基用例可以依赖包含用例执行的结果，但是双方都不能访问对方的属性。
包含关系对典型的应用就是复用，也就是定义中说的情景。但是有时当某用例的事件流过于复杂时，为了简化用例的描述，我们也可以把某一段事件流抽象成为一个被包含的用例；相反，用例划分太细时，也可以抽象出一个基用例，来包含这些细颗粒的用例。这种情况类似于在过程设计语言中，将程序的某一段算法封装成一个子过程，然后再从主程序中调用这一子过程。

例如：业务中，总是存在着维护某某信息的功能，如果将它作为一个用例，那新建、编辑以及修改都要在用例详述中描述，过于复杂；如果分成新建用例、编辑用例和删除用例，则划分太细。这时包含关系可以用来理清关系。

![](/img/use_case_relation.gif)

## 扩展(extend)

扩展关系：将基用例中一段相对独立并且可选的动作，用扩展（Extension）用例加以封装，再让它从基用例中声明的扩展点（Extension Point）上进行扩展，从而使基用例行为更简练和目标更集中。扩展用例为基用例添加新的行为。扩展用例可以访问基用例的属性，因此它能根据基用例中扩展点的当前状态来判断是否执行自己。但是扩展用例对基用例不可见。

对于一个扩展用例，可以在基用例上有几个扩展点。
例如，系统中允许用户对查询的结果进行导出、打印。对于查询而言，能不能导出、打印查询都是一样的，导出、打印是不可见的。导入、打印和查询相对独立，而且为查询添加了新行为。因此可以采用扩展关系来描述：

![](/img/use_case_relation2.gif)

## 泛化(generalization)

泛化关系：子用例和父用例相似，但表现出更特别的行为；子用例将继承父用例的所有结构、行为和关系。子用例可以使用父用例的一段行为，也可以重载它。父用例通常是抽象的。在实际应用中很少使用泛化关系，子用例中的特殊行为都可以作为父用例中的备选流存在。

例如，业务中可能存在许多需要部门领导审批的事情，但是领导审批的流程是很相似的，这时可以做成泛化关系表示：

![](/img/use_case_relation3.gif)