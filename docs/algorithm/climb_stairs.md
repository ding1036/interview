<a id = "jump">[算法题列表](algorithm.md)</a>

描述

假设你正在爬楼梯，需要n步你才能到达顶部。但每次你只能爬一步或者两步，你能有多少种不同的方法爬到楼顶部？

样例
```
Example 1:
	Input:  n = 3
	Output: 3
	
	Explanation:
	1) 1, 1, 1
	2) 1, 2
	3) 2, 1
	total 3.


Example 2:
	Input:  n = 1
	Output: 1
	
	Explanation:  
	only 1 way.
```


解1：
这个题本质就是解裴波拉切数

定义F(n)表示到达第n个台阶的方法，则F(n) = F(n - 1) +F(n - 2) 
非递归
```java
 public class Solution {
    /**
     * @param n: An integer
     * @return: An integer
     */
    public int climbStairs(int n) {
        // write your code here
        if(n<=0)
            return 0;
        if(n == 1)
            return 1;
        if(n == 2)
            return 2;
        //初始化
        int x = 1;
        int y = 2;
        int result = 0;
        while(n>=3) {
            result = x + y;
            x = y;
            y = result;
            n--;
        }
        return result;
    }
}
```

解决2：
递归(太大会溢出)
```java
public static int climbStairs(int n) {
         if(n<=0)
            return 0;
        if(n == 1)
            return 1;
        if(n == 2)
            return 2;
        return  climbStairs(n-1)+climbStairs(n-2);    
}
```
这里可以看出当n很大时递归深度很深，时间复杂度为指数级别，导致超时，这里我们观察到climbStairs(n-2),climbStairs(n-3)…被调用的多次，如果我们可以找个容器将他的值存储下来，下次遇到的时候判断容器里面是否有这个函数的值，如果有直接拿出来用，如果没有计算以后把它放到容器中。

这种算法叫做备忘录算法

备忘录算法：

```java
class Solution {
    Map<Integer,Integer> map = new HashMap<>();
    public int climbStairs(int n) {
        if(n == 1)
            return 1;
        if(n == 2)
            return 2;
        if(map.containsKey(n)){
            return map.get(n);
        }
        int value = climbStairs(n-1) + climbStairs(n-2);
        map.put(n,value);
        return value;
    }
}
```

但是使用备忘录算法算法时间复杂度还是很高，能不能不使用递归，这里我们正向思考

从第一层楼梯可以到达第二层或者第三层，第二层可以到达第三层或者第四层…以此类推
dp[1] 	dp[2] 	dp[3] 	dp[4] 	dp[5] 	…
1 	2 	3 	5 	8 	…

观察得知dp[i] = dp[i-1] + dp[i-2],i>=3

DP:
```java
class Solution {
    public int climbStairs(int n) {
        //dp数组存放的是到达第n层共有多少方法
        int[] dp = new int[n+1];
        dp[1] = 1;
        if(n > 1){
            dp[2] = 2;
        }       
        for(int i = 3;i <= n;++i){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
}
```

此时时间复杂度O(n) 空间复杂度O(n) 我们能不能优化空间？

由观察得：dp[i] = dp[i-1] + dp[i-2] 只与前两个数有关，那么我们没有必要存储所有的数了

优化DP:
```java
class Solution {
    public int climbStairs(int n) {
        if(n == 1)
            return 1;
        //初始值
        int a = 1;
        int b = 2;
        //temp代替数组
        int temp;
        for(int i = 3;i <= n;++i){
            temp = a;
            a = b;
            b = b + temp;
        }
        return b;
    }
}
```
解决3：

```java
class Solution {
    public int climbStairs(int n) {
        if (n<=0) {
            return 0;
        }
        if (n == 1) {
            return 1;
        }
        int[] dp = new int[2];
        dp[0] = 1;
        dp[1] = 1;
        for (int i = 2; i <= n; i++) {
            dp[i % 2] = dp[(i - 1) % 2] + dp[(i - 2) % 2];
        }
        
        return dp[n % 2];
    }
}
```
