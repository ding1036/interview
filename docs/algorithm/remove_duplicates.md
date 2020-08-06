<a id = "jump">[算法题列表](algorithm.md)</a>

描述

给定一个排序数组，在原数组中“删除”重复出现的数字，使得每个元素只出现一次，并且返回“新”数组的长度。

不要使用额外的数组空间，必须在不使用额外空间的条件下原地完成。

样例

样例 1:
```
输入:  []
输出: 0
```
样例 2:
```
输入:  [1,1,2]
输出: 2	
解释:  数字只出现一次的数组为: [1,2]
```

解1：

```java
public class Solution {
    /*
     * @param nums: An ineger array
     * @return: An integer
     */
    public int removeDuplicates(int[] nums) {
        // write your code here
        if(nums==null || nums.length==0){
            return 0;
        } 
        int index=0;
        for(int i=1;i<nums.length;i++){
            if(nums[i]!=nums[index]){
                nums[++index]=nums[i];
            }
        }
        return index+1;
    }
}
```
