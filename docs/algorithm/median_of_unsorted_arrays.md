<a id = "jump">[首页](/README.md)</a>
中位数，就是数组排序后处于数组最中间的那个元素。说来有些麻烦，如果数组长度是奇数，最中间就是位置为（n+1）／2的那个元素。如果是偶数呢，标准的定义是位置为n/2和位置为n/2+1的两个元素的和除以2的结果，有些复杂。

# 快速算法

快速算法是快速中位数算法，类似于快速排序，采用的是分而治之的思想。基本思路是：任意挑一个元素，以该元素为支点，将数组分成两部分，左部分是小于等于支点的，右部分是大于支点的。如果你的运气爆棚，左部分正好是（n－1）／2个元素，那么支点的那个数就是中位数。

```java
public static double median2(int[] array){
    if(array==null || array.length==0) return 0;
    int left = 0;
    int right = array.length-1;
    int midIndex = right >> 1;
    int index = -1;
    while(index != midIndex){
        index = partition(array, left, right);
        if(index < midIndex) left = index + 1;
        else if (index > midIndex) right = index - 1;
        else break;
    }
    return array[index];
}

public static int partition(int[] array, int left, int right){
    if(left > right) return -1;
    int pos = right;
    right--;
    while(left <= right){
        while(left<pos && array[left]<=array[pos]) left++;
        while(right>left && array[right]>array[pos]) right--;
        if(left >= right) break;
        swap(array, left, right);
    }

    swap(array, left, pos);
    return left;
}
```

# 使用最小堆

首先将数组的前（n+1）／2个元素建立一个最小堆。

然后，对于下一个元素，和堆顶的元素比较，如果小于等于，丢弃之，接着看下一个元素。如果大于，则用该元素取代堆顶，再调整堆，接着看下一个元素。重复这个步骤，直到数组为空。

当数组都遍历完了，那么，堆顶的元素即是中位数。

可以看出，长度为（n＋1）／2的最小堆是解决方案的精华之处。

```java
public static double median(int[] array){
    int heapSize = array.length/2 + 1;
    PriorityQueue<Integer> heap = new PriorityQueue<>(heapSize);
    for(int i=0; i<heapSize; i++){
        heap.add(array[i]);
    }
    for(int i=heapSize; i<array.length; i++){
        if(heap.peek()<array[i]){
            heap.poll();
            heap.add(array[i]);
        }
    }
    if(array.length % 2 == 1){
        return (double)heap.peek();
    }
    else{
        return (double)(heap.poll()+heap.peek())/2.0;
    }
}
```

[toTop](#jump)