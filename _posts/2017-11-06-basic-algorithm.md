---
layout : post
title : "几种基础算法的简介"
category : 算法
tags : 冒泡 快速 希尔
---
* content
{:toc}

　　虽然暂时并没有在工作中使用过几种基础算法，但这些知识偶尔还是要温新一下。包括冒泡、快排、插入、希尔和堆排序等，我简单的说明解释一下，使其更容易明白。


### 基础排序算法

　　因为平时的工作很少涉及到这类算法的使用，所以我也只是介绍一下这些算法的思想，给出具体实现的代码，但不会去解释代码。下面介绍的算法有冒泡、快排、插入、选择、归并、堆排序和希尔排序。（为了方便说明，以下排序都都递增排序，提供算法代码实现仅为参考）

#### 冒泡排序

　　冒泡排序是一种非常容易理解的排序算法，从数组第一位开始，和其后面的数进行比较，大于其后面的一位的数，则两者交互位置。例如：[3,2,1]-&gt;[2,3,1]-&gt;[2,1,3]。

```java
//冒泡排序
public class BubbleSort {
    public int[] bubbleSort(int[] A, int n) {
        if(n < 2)
            return A;
        for(int j = n; j>1;j--){
            for(int i = 1; i < j; i++){
	            if(A[i-1] > A[i]){
	                int temp = A[i-1];
	                A[i-1] = A[i];
	                A[i] = temp;
	            }
        	}
        }
        return A;
    }
}
```

#### 选择排序

　　选择排序的思想为选择，每次选出未排序部分的最小（大）值和未排序部分的头（尾）交互，直至所有排序完成。例如：[4,3,2,1]-&gt;[1,3,2,4]-&gt;[1,2,3,4]。

```java
//选择排序
public class SelectionSort {
    public int[] selectionSort(int[] A, int n) {
        int temp;
        for(int i = 0; i < n; i++){
            int min = i;
            for(int j = i+1; j < n; j++){
                if(A[j] < A[min]){
                    min = j;
                }
            }
            if(min != i){
                temp = A[i];
                A[i] = A[min];
                A[min] = temp;
            }
        }
        return A;
    }
}
```

#### 插入排序

　　插入排序的思想为在一个有序的数组序列中找到合适的位置，插入一个新的数，使得插入后的序列仍然有序。例如：[4,3,2,1]-&gt;[3,4,1,2]-&gt;[1,3,4,2]

```java
//插入排序
public class InsertionSort {
    public int[] insertionSort(int[] A, int n) {
        int temp;
        for(int i = 0; i < n-1; i++){
            for(int j = i+1; j > 0; j--){
                if(A[j] < A[j-1]){
                    temp = A[j];
                    A[j] = A[j-1];
                    A[j-1] = temp;
                }
            }
        }
        return A;
    }
}
```

#### 归并排序

　　归并排序的的思想为先分组，再排序，再合并，再排序...。首先是将数组中相邻元素两两分组，排序，然后合并相邻两组，排序。由此看出，合并和排序是一个递归的过程。例如：[4,3,2,1]-&gt;[3,4,1,2]-&gt;[1,2,3,4]。

```java
//归并排序
public class MergeSort {
    public int[] mergeSort(int[] A, int n) {
        if(n < 2)
            return A;
        else{
            int [] b = new int[n];
            b = merge_sort(A, b, n-1, 0);
            return b;
        }
    }
    
    public int[] merge_sort(int[] a, int[] b, int right, int left){
        int middle;
        int [] t = new int[right+1];
        if(right == left)
            b[left] = a[left];
        else{
            middle = (right+left)/2;
            merge_sort(a, t, middle, left);
            merge_sort(a, t, right, middle+1);
            merge(t, b, right, left, middle);
        }
        return b;
    }
    
    public void merge(int[] a, int[] b, int right, int left, int middle){
        int i = left, k = left, j = middle+1;
        while(i <= middle && j <= right){
            if(a[i] <= a[j]){
                b[k++] = a[i++];
            }else b[k++] = a[j++];
        }
        while(i <= middle){
            b[k++] = a[i++];
        }
        while(j <= right){
            b[k++] = a[j++];
        }
    }
}
```

#### 快速排序

　　快速排序的思想为随机选取数组中的一个数（一般选取第一个），将比它小的放它左边，比它大的放它右边，然后按此方法递归其左右数列。例如：[3,2,1,5,4]-&gt;[2,1,3,5,4]-&gt;[1,2,3,4,5]。

```java
//快速排序
public class QuickSort {
    public int[] quickSort(int[] A, int n) {
        if(n > 1){
            quick_sort(A, 0, n-1);
        }
        return A;
    }
    
    public int partition(int[] A, int left, int right){
        int i = left, j = right;
        int temp = A[left];
        while(i < j){
            while(i < j && temp <= A[j]){
                j--;
            }
            if(i < j){
                A[i] = A[j];
                i++;
            }
            while(i < j && temp >= A[i]){
                i++;
            }
            if(i < j){
                A[j] = A[i];
                j--;
            }
        }
        A[i] = temp;
        return i;
    }
    
    public void quick_sort(int[] A, int left, int right){
        int pivot;
        if(left < right){
            pivot = partition(A, left, right);
            quick_sort(A,left, pivot-1);
            quick_sort(A,pivot+1, right);
        }
    }
}
```

#### 堆排序

　　堆排序需要先搞懂如何实现大（小）根堆，利用大（小）根堆能迅速找到数组的最大（小）值，然后通过不断地调整大（小）根堆完成排序。

　　大根堆实际是一棵二叉树，特点是所有的子节点都小于其父节点。然后，因为是二叉树，所以能根据计算公式知道某个节点的父节点和左右子节点。只需要按照子节点小于其父节点的规则进行调整，就可以得到大根堆。

　　得到大根堆之后，就将根节点与最后一个子节点互换，然后移除该子节点，因为该节点是数组的最大值，移除之后，再重新调整大根堆，如此循环之后就能得到有序数组。

```java
//堆排序
public class HeapSort {
    public int[] heapSort(int[] A, int n) {
    	if(A == null || n < 2)
    		return A;
        int temp;
        createHeap(A, n);
        for(int i = n; i > 1; i--){
            //将大顶堆的根结点移除，放在数组末尾
            temp = A[0];
            A[0] = A[i-1];
            A[i-1] = temp;
            adjustHeap(A, 1, i-1);
        }
        return A;
    }
    
    public void createHeap(int[] H, int n){
        for(int i = n/2; i > 0; i--){
            adjustHeap(H, i, n-1);
        }
    }
    
    public void adjustHeap(int[] H, int s, int m){
        int temp = H[s-1];
        for(int j = 2*s; j <= m; j=2*j){
            //沿值大的结点往下筛选
            if(j < m && H[j-1] < H[j])
                j++;
            if(temp > H[j-1])
                break;
            H[s-1] = H[j-1];
            s = j;
        }
        H[s-1] = temp;
    }
}
```

#### 希尔排序

　　希尔排序是插入排序的变化。将原来插入排序的逐个比较变为间隔为n的元素相互比较。然后逐渐减小n的值，完成排序。例如n为3，[9,8,7,6,5,4,3,2,1]-&gt;[3,8,7,6,5,4,9,2,1]。该数组a的a[0]于a[3]进行比较，因为9大于6，因此两元素互换位置，然后a[3]与a[6]进行比较，3小于9，于是互换位置，然后a[3]与a[0]比较，3小于6，于是互换位置。其他元素也是一样与间隔为3的元素比较，最终得到有序数列。

```java
//希尔排序
public class ShellSort {
    public int[] shellSort(int[] A, int n) {
        if(A.length < 2)
            return A;
        for(int i = n/2; i > 0; i--){
            shellInsert(A, i);
        }
        return A;
    }
    
    public void shellInsert(int[] A, int c){
        int temp;
        for(int i = c; i < A.length; i++){ 
            int j = i;
           	while(j >= c && A[j] < A[j-c]){
                temp = A[j];
                A[j] = A[j-c];
                A[j-c] = temp;
                j = j - c;
           	}
        }
    }
}
```