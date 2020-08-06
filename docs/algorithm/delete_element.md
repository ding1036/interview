<a id = "jump">[算法题列表](algorithm.md)</a>

描述

给定一个数组和一个值，在原地删除与值相同的数字，返回新数组的长度。

元素的顺序可以改变，并且对新的数组不会有影响。

样例
```
Example 1:
	Input: [], value = 0
	Output: 0


Example 2:
	Input:  [0,4,4,0,0,2,4,4], value = 4
	Output: 4
	
	Explanation: 
	the array after remove is [0,0,0,2]
```


解1：

```java

public class Solution {
    /*
     * @param A: A list of integers
     * @param elem: An integer
     * @return: The new length after remove
     */
    public int removeElement(int[] A, int elem) {
        // write your code here
    	 int j= 0;
		 for(int i =0;i<A.length;i++){
			 if(A[i]!=elem){
				 A[j]=A[i];
				 j++;
			 }
		 }
			return j;	 
        
    }
}
```