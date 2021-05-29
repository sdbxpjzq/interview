## 快排

平均时间复杂度**O(nlogn)**,  最坏的情况**O(n^2), 不稳定排序**

```java
// 非递归写法
private static void quickSort(int[] arr, int left, int right) {
    // 用栈模拟
    Stack<Integer> stack = new Stack<>();
    if (left < right) {
        stack.push(right);
        stack.push(left);
        while (!stack.isEmpty()) {
            int l = stack.pop();
            int r = stack.pop();
            int pivot = Partition(arr, l, r);
            if (l < pivot - 1) {
                stack.push(pivot - 1);
                stack.push(l);
            }
            if (r > pivot + 1) {
                stack.push(r);
                stack.push(pivot + 1);
            }
        }
    }
}
// 递归写法
private static void quickSortV3(int[] arr, int left, int right) {
    // 是否只剩下一个元素
    if (left >= right) {
        // 递归退出
        return;
    }
    // 基准值
    int pivot = partition(arr, left, right);
    quickSortV3(arr, left, pivot - 1);// left ~ pivot-1 的元素都比小
    quickSortV3(arr, pivot + 1, right);
}
//划分方法
  private  int partition(int[] a, int left, int right) {
    int mid = left;
    
    while(left<right) {
      while(left<right && a[right]>=a[mid]){
        right--;
      }
        
      while(left<right && a[left]<=a[mid]) {
        left++;
      }
       
      if(left<right) {
        swap(a,left,right);
      }
    }
    swap(a,mid,left);
    return left;
  }
  
//交换方法
  private static void swap(int[] a, int left, int right) {
    int t = a[left];
    a[left] = a[right];
    a[right] = t;
  }
```
