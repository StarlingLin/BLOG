---
title: 椋鸟数据结构笔记#7：堆排序、TopK
date: 2024-04-06 23:12:20
category:
  - [计算机科学, 数据结构笔记]
tags:
  - 数据结构
  - 二叉树
  - 堆
  - 排序
math: true
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

#### 堆排序

堆排序，就是利用堆的思想进行排序，是一种非常高效的排序方法。

它的基本思想是将**待排序的序列构建成一个堆**，这样，序列的最大（最小）值就是堆顶的根节点。将其**与堆的最后一个元素交换**，然后将**剩余的 n-1 个元素的序列重新构建成一个堆**，这样就会得到 n 个元素的次大（小）值。如此反复执行，便能得到一个有序序列。

基本步骤如下：

1. **建堆**：将序列构建成堆，如果想要从大到小的序列，就构建大根堆，反正构建小根堆。
2. **交换**：将堆顶与最后一个元素交换，然后==将堆容量减一（原堆顶不再视为堆中元素）==。
3. **向下调整**：对顶元素向下调整，使得新的根节点为新堆的最大值。
4. **重复上述过程直到完全排序**。

这个过程看起来似乎也不简单，但是计算可发现其时间复杂度仅为$O(n\log n)$，空间复杂度仅为$O(1)$。

注意：堆排序是一种不稳定的原地排序算法，==在进行堆顶和堆尾元素的交换时可能会破坏相同元素的相对顺序==。

##### 堆排序的实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

//#define MIN_HEAP
#define MAX_HEAP

#ifdef MIN_HEAP
#define HEAP_COMPARE(a, b) ((a) < (b))
#endif // MIN_HEAP - 排序后由大到小
#ifdef MAX_HEAP
#define HEAP_COMPARE(a, b) ((a) > (b))
#endif // MAX_HEAP - 排序后由小到大

typedef int HPDataType;

void Swap(HPDataType* a, HPDataType* b)
{
	HPDataType tmp = *a;
	*a = *b;
	*b = tmp;
}

void AdjustUp(HPDataType* data, int child)
{
	int parent = (child - 1) / 2;
	while (child > 0)
	{
		if (HEAP_COMPARE(data[child], data[parent]))
		{
			Swap(&data[child], &data[parent]);
			child = parent;
			parent = (child - 1) / 2;
		}
		else
		{
			break;
		}
	}
}

void AdjustDown(HPDataType* data, int size, int parent)
{
	int child = parent * 2 + 1;
	while (child < size)
	{
		if (child + 1 < size && HEAP_COMPARE(data[child + 1], data[child]))
		{
			++child;
		}
		if (HEAP_COMPARE(data[child], data[parent]))
		{
			Swap(&data[child], &data[parent]);
			parent = child;
			child = parent * 2 + 1;
		}
		else
		{
			break;
		}
	}
}

void HeapSort(HPDataType* a, int n)
{
	//建堆
	for (int i = (n - 2) / 2; i >= 0; --i)
	{
		AdjustDown(a, n, i);
	}

	//排序
	for (int i = 0; i < n; ++i)
	{
		Swap(&a[0], &a[n - 1 - i]);
		AdjustDown(a, n - 1 - i, 0);
	}
}

//测试
int main()
{
	printf("请输入要排序的数字个数：");
	int n;
	scanf("%d", &n);
	HPDataType* a = (HPDataType*)malloc(sizeof(HPDataType) * n);
	//文件输入
	FILE* fp = fopen("data.txt", "r");
	if (fp == NULL)
	{
		printf("文件打开失败\n");
		return 0;
	}
	for (int i = 0; i < n; ++i)
	{
		fscanf(fp, "%d", &a[i]);
	}
	fclose(fp);

	HeapSort(a, n);

	//文件输出
	fp = fopen("output.txt", "w");
	if (fp == NULL)
	{
		printf("文件打开失败\n");
		return 0;
	}
	for (int i = 0; i < n; ++i)
	{
		fprintf(fp, "%d\n", a[i]);
	}
	fclose(fp);

	free(a);
	return EXIT_SUCCESS;
}

////生成随机数
//int main()
//{
//	srand((unsigned int)time(NULL));
//	printf("请输入要生成的随机数个数：");
//	int n;
//	scanf("%d", &n);
//	
//	FILE* fp = fopen("data.txt", "w");
//	if (fp == NULL)
//	{
//		printf("文件打开失败\n");
//		return 0;
//	}
//	for (int i = 0; i < n; ++i)
//	{
//		fprintf(fp, "%d\n", rand());
//	}
//	fclose(fp);
//
//	return EXIT_SUCCESS;
//}
```

#### TopK问题

TopK就是求数据集合中前k个最大或者最小的数据。

比较好想到的办法就是排序然后取前面K个值，但是**如果数据量非常大，排序就会变的不太可能**（**无法全部加载到内存**中。

而比较好的方式就是用堆来寻找TopK，其思路如下：

1. 用数据集合中前K个元素来建堆
   + **取前K个最大元素则建小堆**
   + **取前K个最小元素则建大堆**
2. 用剩余的N-K个元素依次与堆顶元素比较，如果不满足大小关系则替换堆顶元素并向下调整。

##### 用堆实现TopK的时间复杂度

对于前K个元素构建堆的过程，时间复杂度是$O(K)$；对于剩余的N-K个元素，每个元素都需要和堆顶元素进行比较，并可能需要插入堆中，这个过程的时间复杂度是$O(\log K)$（因为每次插入或删除堆顶元素都需要进行堆调整）。所以，对于所有的N-K个元素，时间复杂度是$O((N-K) \log K)$。

综合起来，使用**堆实现TopK的总时间复杂度是**$O(K + (N-K) \log K) = O(N \log K)$。

##### TopK问题的实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MIN_HEAP
//#define MAX_HEAP

#ifdef MIN_HEAP
#define HEAP_COMPARE(a, b) ((a) < (b))
#endif // MIN_HEAP - 寻找前k个最大值
#ifdef MAX_HEAP
#define HEAP_COMPARE(a, b) ((a) > (b))
#endif // MAX_HEAP - 寻找前k个最小值

void AdjustDown(int* arr, int size, int parent)
{
	int child = parent * 2 + 1;
	while (child < size)
	{
		if (child + 1 < size && HEAP_COMPARE(arr[child + 1], arr[child]))
		{
			++child;
		}
		if (HEAP_COMPARE(arr[child], arr[parent]))
		{
			int temp = arr[child];
			arr[child] = arr[parent];
			arr[parent] = temp;
			parent = child;
			child = parent * 2 + 1;
		}
		else
		{
			break;
		}
	}
}

void CreateNDate()
{
	// 造数据
	int n = 10000;
	srand((unsigned int)time(NULL));
	const char* file = "data.txt";
	FILE* fin = fopen(file, "w");
	if (fin == NULL)
	{
		perror("fopen error");
		return;
	}

	for (size_t i = 0; i < n; ++i)
	{
		int x = rand();
		fprintf(fin, "%d\n", x);
	}

	fclose(fin);
}

void PrintTopK(int k)
{
	FILE* fin = fopen("data.txt", "r");
	if (fin == NULL)
	{
		perror("fopen error");
		return;
	}
	int val = 0;
	int* arr = (int*)malloc(sizeof(int) * k);
	if (arr == NULL)
	{
		perror("malloc error");
		return;
	}
	for (int i = 0; i < k; ++i)
	{
		fscanf(fin, "%d", &val);
		arr[i] = val;
	}
	//前k个数建堆
	for (int i = (k - 2) / 2; i >= 0; --i)
	{
		AdjustDown(arr, k, i);
	}
	int x = 0;
	while (fscanf(fin, "%d", &x) != EOF)
	{
		if (!HEAP_COMPARE(x, arr[0]))
		{
			arr[0] = x;
			AdjustDown(arr, k, 0);
		}
	}
	for (int i = 0; i < k; ++i)
	{
		printf("%d ", arr[i]);
	}
}

int main()
{
	int k = 0;
	printf("请输入要找的前k个数：");
	scanf("%d", &k);
	CreateNDate();
	PrintTopK(k);
	return 0;
}
```

