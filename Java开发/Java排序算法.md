#Java排序算法

###排序算法的分类
1. 插入排序：直接插入排序和希尔排序
2. 选择排序：简单选择排序和堆排序
3. 交换排序：冒泡排序和快速排序
4. 归并排序
5. 基数排序

###各种排序算法的比较

|排序方法 |时间复杂度(平均)|空间复杂度|稳定性
|:--|:--|:--|:--|
|直接插入排序|O(n²)|O(1)|稳定|
|希尔排序|O(n²)时间复杂度与增量序列有关|O(1)|不稳定|
|简单选择排序|O(n²)|O(1)|不稳定|
|堆排序|O(nlogn)|O(1)|不稳定|
|冒泡排序|O(n²)|O(1)|稳定|
|快速排序|O(nlogn)|O(nlogn)|不稳定|
|归并排序|O(nlogn)|O(n)|稳定|
|基数排序|O(d(r+n))|O(rd+n)|稳定|
|基数排序的复杂度|r代表关键字的基数|d代表长度|n代表关键字的个数|

###直接插入排序
- 原理：将一个记录插入到已经排好的有序表中。
- 解析：先把第一个记录当做是有序的子序列，然后将第二个记录逐个进行插入，直到整个序列有序。需要进行N-1次插入

```
public static <T extends Comparable<? super T>> void insertSort(T[] arrays) {
        if (arrays == null || arrays.length <= 1) {
            return;
        }

        for (int i = 1; i < arrays.length; i++) {
            T temp = arrays[i];
            int j;
            for (j = i; j > 0 && temp.compareTo(arrays[j - 1]) < 0; j--) {
                arrays[j] = arrays[j - 1];
            }
            arrays[j] = temp;
        }
    }
```

###希尔排序
- 原理：比较相距一定间隔的元素来工作，各趟比较所用的距离随着算法的进行而减少，直到只比较相邻元素的最后一趟排序为止
- 分析：
  1. 确认希尔排序的步长h=arrays.length / 2，假设数组的长度为10，那么步长为5、2、1
  2. 步长为5时，比较第六个位置的数和第一个位置的数，小的在前大的再后。直到比较到第十个位置为止
  3. 步长为2时，比较第三个位置的数和第一个位置的数，小的在前大的再后。假如依次到了第五个位置的数和第三个位置的数比较，小的移到第三个位置，然后第三个位置与第一个位置比较。
  4. 步长为1时，基本上数组已经是有序的。然后依次比较相邻的元素。

```
public static <T extends Comparable<? super T>> void shellSort(T[] arrays) {
        if (arrays == null || arrays.length <= 1) {
            return;
        }

        //希尔排序步长
        int incrementNum = arrays.length / 2;
        while (incrementNum >= 1) {
            for (int i = incrementNum; i < arrays.length; i++) {
                T temple = arrays[i];
                int j;
                for (j = i; j >= incrementNum && temple.compareTo(arrays[j - incrementNum]) < 0; j -= incrementNum) {
                    arrays[j] = arrays[j - incrementNum];
                }
                arrays[j] = temple;
            }
            //设置新的增量
            incrementNum = incrementNum / 2;
        }
    }
```

###简单选择排序
- 原理：进行N-1次选择，从数组中选择最小或者最大的与第N个位置的数交换位置。
- 分析：每次遍历得到最小元素的下标，然后将元素与指定的位置交换。如第一次遍历就是把最小的元素和第一个位置的元素交换。

```
public static <T extends Comparable<? super T>> void selectSort(T[] arrays) {
        if (arrays == null || arrays.length <= 1) {
            return;
        }

        int length = arrays.length;
        for (int i = 0; i < length; i++) {
            int k = i;//存放最小值下标,每次循环最小值的下标+1
            for (int j = i; j < length; j++) {
                if (arrays[k].compareTo(arrays[j]) > 0) {
                    k = j;
                }
            }
            //交换,把最小的值放在指定位置
            if (i != k) {
                T temple = arrays[i];
                arrays[i] = arrays[k];
                arrays[k] = temple;
            }
        }

        printArrays(arrays);
    }
```

###堆排序
- 原理：先建堆，然后删除
- 分析

```
```

###冒泡排序
- 原理：比较相邻的元素，小的在前大的在后。
- 分析：如果第一个比第二个大，就交换他们两个。一趟结束，最后的元素一定是最大的。

```
public static <T extends Comparable<? super T>> void bubbleSort(T[] arrays) {
        if (arrays == null || arrays.length <= 1) {
            return;
        }

        int length = arrays.length;
        for (int i = 0; i < length - 1; i++) {
            for (int j = 0; j < length - 1 - i; j++) {
                if (arrays[j].compareTo(arrays[j + 1]) > 0) {
                    T temple = arrays[j];
                    arrays[j] = arrays[j + 1];
                    arrays[j + 1] = temple;
                }
            }
        }
    }
```

###快速排序
- 原理：利用递归的思想，进行交换排序
- 分析：

```
public static <T extends Comparable<? super T>> void quickSort(T[] arrays) {
        quickSort(arrays, 0, arrays.length - 1);

        printArrays(arrays);
    }

    private static <T extends Comparable<? super T>> void quickSort(T[] a, int low, int high) {
        if (low < high) { //如果不加这个判断递归会无法退出导致堆栈溢出异常
            int middle = getMiddle(a, low, high);
            quickSort(a, 0, middle - 1);//递归对低子表递归排序
            quickSort(a, middle + 1, high);//递归对高子表递归排序
        }
    }

    private static <T extends Comparable<? super T>> int getMiddle(T[] a, int low, int high) {
        T key = a[low];//基准元素，排序中会空出来一个位置
        while (low < high) {
            //从high开始找比基准小的，与low换位置
            while (low < high && a[high].compareTo(key) >= 0) {
                high--;
            }
            a[low] = a[high];

            //从low开始找比基准大,放到之前high空出来的位置上
            while (low < high && a[low].compareTo(key) <= 0) {
                low++;
            }
            a[high] = a[low];
        }
        //此时low=high 是基准元素的位置，也是空出来的那个位置
        a[low] = key;
        return low;
    }
```

###归并排序
- 原理：合并两个已经排序好的表，该算法是经典的分治策略，它将问题分成一些小的问题然后递归求解。
- 分析：

```
public static <T extends Comparable<? super T>> void mergeSort(T[] arrays) {
        mergeSort(arrays, 0, arrays.length - 1);
    }

    private static <T extends Comparable<? super T>> void mergeSort(T[] arrays, int left, int right) {
        if (left < right) {
            int mid = (left + right) / 2;
            mergeSort(arrays, left, mid);
            mergeSort(arrays, mid + 1, right);
            mergeSort(arrays, left, mid, right);
        }
    }

    private static <T extends Comparable<? super T>> void mergeSort(T[] arrays, int left, int mid, int right) {
        T[] temp = (T[]) new Comparable[right - left + 1];
        int i = left;// 左指针
        int j = mid + 1;// 右指针
        int k = 0;
        // 把较小的数先移到新数组中
        while (i <= mid && j <= right) {
            if (arrays[i].compareTo(arrays[j]) < 0) {
                temp[k++] = arrays[i++];
            } else {
                temp[k++] = arrays[j++];
            }
        }
        // 把左边剩余的数移入数组
        while (i <= mid) {
            temp[k++] = arrays[i++];
        }
        // 把右边边剩余的数移入数组
        while (j <= right) {
            temp[k++] = arrays[j++];
        }
        // 把新数组中的数覆盖arrays数组
        for (int k2 = 0; k2 < temp.length; k2++) {
            arrays[k2 + left] = temp[k2];
        }

    }
```

###基数排序
- 原理：基于桶的思想,进行最低位优先法或者最高位优先法(计算机用最低位优先法方便),依次比较对应位的大小
- 分析：

```
public static void radixSort(int[] data, int radix, int d) {
        // 缓存数组
        int[] tmp = new int[data.length];
        // buckets用于记录待排序元素的信息
        // buckets数组定义了max-min个桶
        int[] buckets = new int[radix];

        for (int i = 0, rate = 1; i < d; i++) {
            // 重置count数组，开始统计下一个关键字
            Arrays.fill(buckets, 0);
            // 将data中的元素完全复制到tmp数组中
            System.arraycopy(data, 0, tmp, 0, data.length);

            // 计算每个待排序数据的子关键字
            for (int j = 0; j < data.length; j++) {
                int subKey = (tmp[j] / rate) % radix;
                buckets[subKey]++;
            }

            for (int j = 1; j < radix; j++) {
                buckets[j] = buckets[j] + buckets[j - 1];
            }

            // 按子关键字对指定的数据进行排序
            for (int m = data.length - 1; m >= 0; m--) {
                int subKey = (tmp[m] / rate) % radix;
                data[--buckets[subKey]] = tmp[m];
            }
            rate *= radix;
        }
    }
```

