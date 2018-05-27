<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [是否允许空值](#是否允许空值)
- [是否有序](#是否有序)
- [ArrayList和LinkedList区别](#arraylist和linkedlist区别)
    - [时间复杂度](#时间复杂度)
- [HashMap](#hashmap)

<!-- /TOC -->
# 是否允许空值
* **List** 集合可以存储多个null；
* **Set**集合也可以存储null，但只能存储一个；
* **HashMap**可以存储null键值对，键和值都可以是null，但如果添加的键值对的键相同，则后面添加的键值对会覆盖前面的键值对，即之后存储后添加的键值对；
* **Hashtable**不能添加null，抛空指针    
[toTop](#jump)

# 是否有序
![](/img/collection_order.png)
除了set不可重复，其余均可 map KEY也不可，value可以。除了list和tree有序，其余均无序。   
[toTop](#jump)

# ArrayList和LinkedList区别
1) ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。 
2) 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。 
3) 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。
## 时间复杂度 
**ArrayList** 是线性表（数组）    
**get()** 直接读取第几个下标，复杂度 O(1)    
**add(E)** 添加元素，直接在后面添加，复杂度O（1）   
**add(index, E)** 添加元素，在第几个元素后面插入，后面的元素需要向后移动，复杂度O（n）        
**remove()** 删除元素，后面的元素需要逐个移动，复杂度O（n）    

**LinkedList** 是链表的操作   
**get()** 获取第几个元素，依次遍历，复杂度O(n)   
**add(E)** 添加到末尾，复杂度O(1)    
**add(index, E)** 添加第几个元素后，需要先查找到第几个元素，直接指针指向操作，复杂度O(n)    
**remove()** 删除元素，直接指针指向操作，复杂度O(1)      
[toTop](#jump)

# HashMap

[toTop](#jump)