<a id = "jump">[算法题列表](algorithm.md)</a>


描述

给定一个整型数组，找出主元素，它在数组中的出现次数严格大于数组元素个数的二分之一。

样例

样例 1:
```
输入: [1, 1, 1, 1, 2, 2, 2]
输出: 1
```
样例 2:
```
输入: [1, 1, 1, 2, 2, 2, 2]
输出: 2
```
挑战

要求时间复杂度为O(n)，空间复杂度为O(1)


解1：

```java

public class Solution {
    /*
     * @param nums: a list of integers
     * @return: find a  majority number
     */
    public int majorityNumber(List<Integer> nums) {
        // write your code here
        HashMap<Integer,Integer> map=new HashMap<Integer, Integer>();
    	map.put(nums.get(0), 1);
		for(int i=1;i<nums.size();i++){
			if(map.containsKey(nums.get(i))){
				map.put(nums.get(i), map.get(nums.get(i))+1);
			}else{
				map.put(nums.get(i), 1);
			}
		}
		Set<Integer> set=map.keySet();
		Iterator<Integer> it=set.iterator();
		while(it.hasNext()){
			int temp=it.next();
			if(map.get(temp)>nums.size()/2){
				return temp;
			}
		}
		return 0;
    }
}
```