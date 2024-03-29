<a id = "jump">[首页](/README.md)</a>


# 用例图中三种关系

## 包含(include)
包含是指当多个用例中存在相同的事件流时，能够把这些公共事件流抽象成公共用例，这个公共用例称之为抽象用例（跟类的概念有点相像，类是多个对象的抽象定义），而原始用例称为基础用例，基础用例与抽象用例之间就是包含关系。可是值得注意的是，对于包含关系而言，基础用例是抽象用例执行中不可缺乏的一部分，基础用例通常不单独存在且基础用例不知道抽象用例的存在而抽象用例知道基础用例的存在。包含关系是箭头从抽象用例指向基础用例（也就是从父类指向子类）。

包含关系：使用包含（Inclusion）用例来封装一组跨越多个用例的相似动作（行为片断），以便多个基（Base）用例复用。基用例控制与包含用例的 关系，以及被包含用例的事件流是否会插入到基用例的事件流中。基用例可以依赖包含用例执行的结果，但是双方都不能访问对方的属性。
包含关系对典型的应用就是复用，也就是定义中说的情景。但是有时当某用例的事件流过于复杂时，为了简化用例的描述，我们也可以把某一段事件流抽象成为一个被包含的用例；相反，用例划分太细时，也可以抽象出一个基用例，来包含这些细颗粒的用例。这种情况类似于在过程设计语言中，将程序的某一段算法封装成一个子过程，然后再从主程序中调用这一子过程。

例如：业务中，总是存在着维护某某信息的功能，如果将它作为一个用例，那新建、编辑以及修改都要在用例详述中描述，过于复杂；如果分成新建用例、编辑用例和删除用例，则划分太细。这时包含关系可以用来理清关系。

![](/img/use_case_relation.gif)

## 扩展(extend)
若是一个用例明显地混合了两种或两种以上不一样的场景，能够将这个用例分为一个基础用例和一个扩展用例。扩展关系用<<extend>>关系表示，箭头指向基本用例（也就是从子类指向父类）。与此同时，扩展用例是基础用例在某些特定条件下触发产生的，扩展用例不是基础用例必须存在的部分，扩展用例能够单独存在，扩展用例知道基础用例的存在而基础用例不知道基础用例的存在。

扩展关系：将基用例中一段相对独立并且可选的动作，用扩展（Extension）用例加以封装，再让它从基用例中声明的扩展点（Extension Point）上进行扩展，从而使基用例行为更简练和目标更集中。扩展用例为基用例添加新的行为。扩展用例可以访问基用例的属性，因此它能根据基用例中扩展点的当前状态来判断是否执行自己。但是扩展用例对基用例不可见。

对于一个扩展用例，可以在基用例上有几个扩展点。
例如，系统中允许用户对查询的结果进行导出、打印。对于查询而言，能不能导出、打印查询都是一样的，导出、打印是不可见的。导入、打印和查询相对独立，而且为查询添加了新行为。因此可以采用扩展关系来描述：


![](/img/use_case_relation2.gif)

## 泛化(generalization)
泛化关系是一种继承关系，子用例将继承基用例的全部行为，也就是任何使用基用例的地方均可以使用子用例来代替。我平时是这样记住这个关系的，就是子类从父类中继承，父类就是子类的泛化。由于泛化和继承本就是一对反关系。泛化关系在用例图中用空心箭头表示，箭头方向从子用例指向基用例。

泛化关系：子用例和父用例相似，但表现出更特别的行为；子用例将继承父用例的所有结构、行为和关系。子用例可以使用父用例的一段行为，也可以重载它。父用例通常是抽象的。在实际应用中很少使用泛化关系，子用例中的特殊行为都可以作为父用例中的备选流存在。

例如，业务中可能存在许多需要部门领导审批的事情，但是领导审批的流程是很相似的，这时可以做成泛化关系表示：

![](/img/use_case_relation3.gif)