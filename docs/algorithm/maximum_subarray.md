<a id = "jump">[算法题列表](algorithm.md)</a>

描述

给定一个整数数组，找到一个具有最大和的子数组，返回其最大和。

子数组最少包含一个数


样例

样例1:
```
输入：[−2,2,−3,4,−1,2,1,−5,3]
输出：6
解释：符合要求的子数组为[4,−1,2,1]，其最大和为 6。
```
样例2:
```
输入：[1,2,3,4]
输出：10
解释：符合要求的子数组为[1,2,3,4]，其最大和为 10。
```
挑战

要求时间复杂度为O(n)


解1：

我们先来求出sum数组。sum[i]为从前i项和。

原数组：[-2,  1,  -3,  4,  -1,  2,  1,  -5,  4]

累加和：[-2, -1,  -4,  0,  -1,  1,  2,  -3,  1]

现在的问题是，前面找一个数i，后面找一个数j，使得两数之差sum[j]-sum[i]最大。这就类似于121题，求股票的最大收益，前面找一个值买入，后面找一个值卖出，然后求最大收益。

不过这里有点不一样，就是如果都是正数，如3,2,4,8,1。最好的结果为8，不是8-2=6。所以，如果前面的数是正数，减去一个正数会变小，不如不减。只有前面是负数的时候，减去一个负数会变大。因此，可以设初试的min=0。

 

dp式子：

前i项的最大值 = max{前i-1项的最大值，sum[i]-min}


```java
public class Solution {
    /**
     * @param nums: A list of integers
     * @return: A integer indicate the sum of max subarray
     * 求和最大的一段nums[a]+...+nums[b]最大，即sum[b] - sum[a-1]最大。
     */
    public int maxSubArray(int[] nums) {
        // write your code here
        int n=nums.length;
        for(int i=1;i<n;i++){
            nums[i]+=nums[i-1];
        }
        
        int min = 0;
        int dp=nums[0];
        
        for (int i=0;i<n;i++){
            dp=Math.max(dp,nums[i]-min);
            if(nums[i]<min) min=nums[i];
        }
        
        return dp;
        
    }
}
```