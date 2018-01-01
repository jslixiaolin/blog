# 归并排序之java实现

毕业季在即，一大波面试接踵而来，为了一份好offer，所以最近在重新刷数据结构与算法。正好在博客里面记录一下。今天是归并排序的实现。

归并排序是典型的分治模式的实现，对一个数组A，采取三步实现：分解，解决，合并:
* 分解：分解待排序的n个元素的序列成各具n/2个元素的两个子序列。
* 解决：使用归并排序递归地排序两个子序列。
* 合并：合并两个已排序的子序列产生最后排好序的序列。

那么到底什么是归并排序呢？
归并排序的就是将两个已经排好序的序列进行合并，使得合并后的序列仍是有序的。因为在迭代中，序列中最开始只有一个元素，那么它显然是有序的，进行归并后，在有序的基础上仍然是有序的。

举个例子：假设桌面上有两堆面朝上的扑克牌，都已排好序。最小的牌在顶部。我们希望把两堆牌合并成一堆排序好的牌，牌面朝下的放在桌子上。那么我们自然想到这么做：取两堆牌顶上的较小的那张牌，放到桌子上（这时候取完牌的那堆牌已经露出了一张新的牌）。按照这个步骤继续取牌。直到有一堆牌为空为止。这时候我们只需要把剩下的牌一张张的放桌子上就好了。所以基本需要常量时间，最多n个步骤，所以合并牌需要O(n)的时间。

可能有人会想到，什么时候知道有堆牌为空呢？难道需要每次拿牌的时候都去判断是否为空吗？并不需要这样做，在下面的伪代码中可以看到使用了哨兵的概念，哨兵取为无穷大，结果每当显露一张值为无穷大的牌的时候，显然进入比较的If语句里面，它不可能为较小的牌，除非两堆最顶上的牌都是无穷大，但是这种情况时，所有的非哨兵牌都已经放到桌子上了。已经放了r-p+1张牌，这个时候循环已经结束了。
归并部分的伪代码如下：

```
MERGE(A,p,q,r)
  n1=q-p+1;
  n2=r-q;
  let L[1...n1+1] and R[1...n2+1]
  for i=1 to n1
     L[i]=A(p+i-1);
  for j=1 to n2
     R[j]=A(q+j)
  L[n1+1]=∞
  R[n2+1]=∞
  i=1
  j=1
  for k=p to r
    if L[i]<=R[j]
      A[k]=L[i]
      i=i+1
    else
      A[k]=R[j]
      j=j+1

```
可以看到有这么一行L[n1+1]=∞，这里L[n1+1]=∞ ,这里使用了哨兵的概念。这样是为了避免在每个基本步骤都需要去检车是否有堆为空，节省了效率
整个归并排序的复杂度为：O(nlgn)
 Java实现如下：


```
private static void mergeSentry(int[] source, int begin, int middle, int end) {
    // TODO Auto-generated method stub
    int len1=middle-begin+1;
    int len2=end-middle;
    int[] array1=Arrays.copyOfRange(source,begin,middle+1);
    int[] array2=Arrays.copyOfRange(source, middle+1, end+1);
    int i = 0,j=0;
    for(int k=begin;k<=end;k++){
      if(i<len1&&j<len2){
        if(array1[i]<array2[j]||array1[i]==array2[j]){
          source[k]=array1[i];
          i++;
          continue;
        }
        if(array1[i]>array2[j]){
          source[k]=array2[j];
          j++;
          continue;
        }
      }
      if(i>=len1&&j<len2){
        source[k]=array2[j];
        j++;
        continue;
      }
      if(j>=len2&&i<len1){
        source[k]=array1[i];
        i++;
        continue;
      }
    }
  }
```

完整的伪代码：

```
MERGE-SORT(A,p,r)
 if p<r
   q=(p+r)/2
   MERGE-SORT(A,p,q)
   MERGE-SORT(A,p+1,r)
   MERGE(A,p,q,r)

MERGE(A,p,q,r)
  n1=q-p+1;
  n2=r-q;
  let L[1...n1+1] and R[1...n2+1]
  for i=1 to n1
     L[i]=A(p+i-1);
  for j=1 to n2
     R[j]=A(q+j)
  L[n1+1]=∞
  R[n2+1]=∞
  i=1
  j=1
  for k=p to r
    if L[i]<=R[j]
      A[k]=L[i]
      i=i+1
    else
      A[k]=R[j]
      j=j+1

```

完整的java代码：

```
public class MergeSort {
  public static void main(String[] args) {
    long begin=System.currentTimeMillis();
    int[] a=new int[]{1,22,2,12,120,9,87};
    mergerSort(a, 0, a.length-1);
    System.out.println(Arrays.toString(a));
  }
  public static void mergerSort(int[] source,int begin,int end){
    int middle;
    if(begin<end){
      middle=(int) (Math.floor((begin+end))/2);
      mergerSort(source, begin, middle);
      mergerSort(source, middle+1, end);
      mergeSentry(source,begin,middle,end);
    }
  }

  private static void mergeSentry(int[] source, int begin, int middle, int end) {
    // TODO Auto-generated method stub
    int len1=middle-begin+1;
    int len2=end-middle;
    int[] array1=Arrays.copyOfRange(source,begin,middle+1);
    int[] array2=Arrays.copyOfRange(source, middle+1, end+1);
    int i = 0,j=0;
    for(int k=begin;k<=end;k++){
      if(i<len1&&j<len2){
        if(array1[i]<array2[j]||array1[i]==array2[j]){
          source[k]=array1[i];
          i++;
          continue;
        }
        if(array1[i]>array2[j]){
          source[k]=array2[j];
          j++;
          continue;
        }
      }
      if(i>=len1&&j<len2){
        source[k]=array2[j];
        j++;
        continue;
      }
      if(j>=len2&&i<len1){
        source[k]=array1[i];
        i++;
        continue;
      }
    }
  }
}
```
