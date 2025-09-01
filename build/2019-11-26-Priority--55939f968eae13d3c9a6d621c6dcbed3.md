---
title: Priority Queue and Heap
date: 2019-11-26
tags:
- Cornell
- CS2112
---

---

## Priority Queue

```java
interface PriorityQueue<E> {
    
    void increasePriority(E x, int priority)
        
    //constructor 
	PriorityQueue (ElemOps<E> ops);
}
```



## Binary Heap

### Invariant

1. **Order**: `n.priority<=n.child.priority`  (小根堆)
2. **shape**: 
   - All leaves are at depth h or h-1
   - All h-depth are on the left
   - This heap is in fact a complete binary tree 

### Implementation

#### Swim

if parents have bigger priority, this node should swim. 

#### Sink

if any of the child has higher priority (smaller value), we swap this node with the one with a higher priority (the smaller one). And we only sink when there is actually something to sink (we are at the second deepest level, one level above the leaves)

但是要注意，有的时候你想要sink，但是你的孩子实际上是未被初始化的一些有着随机优先值的结点，这时如果你做交换了，那就全完了。所以应当判断这个孩子结点原本在不在堆中

#### Add

We should preserve both the invariant when adding one element. 

- shape: Ifwe add it at the end of the heap, shape invariant is preserved
- order: This invariant can be broken (though it's at the end, it may not be the node with lowest priority) . So this newly added element should swim. 

#### Extract

extract the head of the heap. But we need to replace it with something to preserve the invariant. 

- shape: The last element in the array is a good candidate. We move it to the root of the tree. This reestablishes the shape invariant
- order: This invariant may now be broken. We fix the heap invariant by sinking the root

要注意我们做任何删除操作的时候，要确保这个元素已经不在数组中了(1. 长度-- 2. 原本的index设为LOWEST_PRIORITY 确保它永远在堆底待着)

#### Change in Priority

- shape: maintained
- order: we may need to bubble the element up if we increase the priority (which for a min-queue means decreasing the value) or down if we decrease the priority.

You need to know the node's index to perform swimming and sinking. But if the node is an object, how would you know its index? We can store its index inside the object. 

#### Heapify

Here is a very efficient way to turn an arbitrary array into a heap. 

> Heapifying can be done by bubbling every element down, starting from the last element in the second last layer (叶子的前一层，叶子那一层没有可以sink的东西，或者说，叶子那一层的Heap Invariant是绝对被保存的，第一个Heap Invariant不被保存的节点是倒数第二层的第一个节点) in the array representation and working backward.
>
> The total time required to do this is linear. At most half the elements need to be bubbled down one step (倒数第二层，叶子前一层), at most a quarter of the elements need to be bubbled down two steps (倒数第三层，叶子前两层), and so on. 

Time Complexity: $n/2 + 2n/4 + 3n/8 + 4n/16 + ... + k^n/2k + ... = 2n.$

This method tells us to start from the end and bubble down. However, we can also start from the front and bubble up. The only difference is that up is slower than down in total (think about how many nodes have to go through how many layers to get to the correct position for each kind of heapify). 

We can also try bubbling down from the front or bubbling up from the end. These two methods are incorrect. Because when we bubble up from the front, we can guarantee that the nodes we have looped through and its parent can form a heap. Similarly, when we bubble down from the end, we can guarantee that the nodes we have looped through and its children can form a heap. However, for bubbling down from the front and bubbling up from the end, no such qualities can be guaranteed. 

#### Resizable Array

You can store your heap in a resizable array with a load factor as we've seen in the HashTables.

#### Code in C++

这份代码有问题，洛谷的数据太弱了没测出来。具体是测试某节点的孩子是否存在的代码错误：堆并不是一个每一层都满的二叉树。正确实现见 [清华DSA 2-7]()

```cpp
#include<iostream>
#include<cstdio>
#include<cmath>
#include<chrono>
#include<time.h>
using namespace std;
const int M=1000000,INF=10e9; int n=0,a[M];
void swim(int); void sink(int);void add(int); int extract(); int head(); void print(); int height(int);
int main()
{
    /* 堆排序小测试
    int num; cin>>num;
    for(int i=0;i<num;i++) {int now; cin>>now; add(now);}
    for(int i=0;i<num;i++) cout<<"extracted"<<extract()<<endl<<endl;*/

    /* 洛谷P3378 模板 堆
    int result[M/2],ans=0; int num; cin>>num;
    for(int i=0;i<num;i++){
        int command; cin>>command;
        switch(command){
            case 1:
                int adding; cin>>adding; add(adding);
                break;
            case 2:
                result[ans++]=head(); break;
            case 3:
                extract(); break;
        }
    }
    for(int i=0;i<ans;i++) cout<<result[i]<<endl;*/

    /* heapify 时间测试
    理论上来说sink比swim要快(需要的操作少)
    但是大概因为我的sink的实现方法太烂，所以竟然 swim 快一点
    int max = 100000;n=max;
    for(int i=0;i<max;i++) a[i] = max-i;

    auto t0 = std::chrono::system_clock::now();

    for (int i = (n/2); i > 0; i--) sink(i);
    //for(int i=2;i<=n;i++) {swim(i);}

    auto t1 = std::chrono::system_clock::now();
    using milliseconds = std::chrono::duration<double, std::milli>;
    milliseconds ms = t1 - t0;
    std::cout << " time taken by my code: " << ms.count() << '\n';*/

    return 0;

}
void swim(int i)
{
    while(a[i/2]>=a[i]&&i>1){
        //注意边界的选择：i<=1的话上面已经没东西了，就没有再上游的必要了
        swap(a[i/2],a[i]);
        i = i/2;
    }
}
void sink(int i)
{
    int l=i*2, r=i*2+1;
    while((a[i]>=a[l]||a[i]>=a[r])&&i<pow(2,height(n))){//叶子的上一层才需要下沉，叶子那一层没有东西可以沉
        if(a[l]<=a[r]&&l<=n){
            //l<=n判断这个孩子结点确实在堆中，而不是一个堆外但在数组中的结点
            //有问题，这个地方如果不把每个删掉的元素设为INF的话，r会一直存在在数组中；
            //考虑2 1 (0) 的情况，其中0是已经被删掉的元素，但是因为a[r]=0<a[l]=1所以2不会sink
            //如果将数组初始化成INF的话，就不需要在不在堆内l<=n这个条件判断了
            //但是内存占用会极大提高，因为整个长为M的数组都被初始化了
            swap(a[i],a[l]);
            i = l; l=i*2; r=i*2+1; continue;
        }
        else if (a[l]>a[r]&&r<=n){
            swap(a[i],a[r]);
            i = r; l=i*2; r=i*2+1; continue;
        }
        else return;
    }
}
void add(int i)
{
    a[++n] = i;
    swim(n);
}
int extract()
{
    int result = a[1];
    swap(a[1],a[n]);
    a[n--]=INF;
    sink(1);
    return result;
}
int head(){return a[1];}

//  returns the height of a complete binary tree with n nodes
int height(int x){
     if(x==0) return 1;
     int digit=0; while(x>0) {x=x>>1; digit++;} return digit;
}

void print(){for(int i=1;i<=n;i++) cout<<a[i]<<" "; cout<<endl;}

```

#### [0-1 Based](https://www.cnblogs.com/yrbbest/p/5290262.html)

上面是用Binary heap设计一个 Max-oriented Priority Queue， 数组是1-based。  假如遇到面试官问怎么heapify怎么办？  下面我们就对上面代码进行少许改动，变为0-based，可以直接对数组进行max -  heapify。

1. heapify()方法: 可以看出我们的heapify方法基本没有变化，除了把N / 2变成了数组的长度 nums.length / 2
2. sink()方法 : 这里我们要注意一下边界条件。 先设置len = nums.length，这里len就相当于之前的N， 然后再进行比较的时候，我们要把每次的 j 都减1，从1-based改变为 0-based，其他代码都不需要改变

```java
public static void heapify(int[] nums) {
        if (nums == null) {
            return;
        }
        for (int k = nums.length / 2; k >= 1; k--) {
            sink(nums, k);
        }
    }
    
    private static void sink(int[] nums, int k) {
        int len = nums.length;
        while (2 * k <= len) {
            int j = 2 * k;
            if (j < len && nums[j - 1] < nums[j]) {
                j++;
            }
            if (nums[k - 1] > nums[j - 1]) {
                break;
            }
            swap(nums, k - 1, j - 1);
            k = j;
        }
    }
```

