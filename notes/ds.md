Bubble Sort (basic idea, O(n²))

Selection Sort

Insertion Sort

Merge Sort (divide & conquer, O(n log n))

Quick Sort (partition + recursion)

Heap Sort (heapify + extract max/min)

Heapify (building heap in O(n))

Insert in Heap (O(log n))

Delete from Heap (O(log n))

Binary Search

Fibonacci (DP + recursion + memoization)

0/1 Knapsack

Euclidean Algorithm for GCD

Sieve of Eratosthenes (prime generation)

bubble sort -
```cpp
for (int i = 0; i < n; i++) {
  for (int j = 0; j < n - 1; j++) {
    if (arr[j] > arr[j + 1]) {
      swap(arr[j], arr[j + 1]);
    }
  }
}
```

selection sort - 
```cpp
for (int i = 0; i < n; i++) {
  int minIdx = i;
  for (int j = i + 1; j < n; j++) {
    if (arr[j] < arr[minIdx]) minIdx = j;
  }
  swap(arr[i], arr[minIdx]);
}
```

insertion sort -
```cpp
for (int i = 1; i < n; i++) {
  int j = i - 1;
  while (j >= 0 && arr[j + 1] < arr[j]) {
    swap(arr[j + 1], arr[j]);
    j--;
  }
}
```

quick sort -
```cpp
int partition(int arr[], int l, int h) {
    int pivot = arr[l];
    int i = l - 1;
    int j = h + 1;

    while (true) {
        // move i right until we find element >= pivot
        do {
            i++;
        } while (arr[i] < pivot);

        // move j left until we find element <= pivot
        do {
            j--;
        } while (arr[j] > pivot);

        if (i >= j)
            return j; // partition index

        swap(arr[i], arr[j]);
    }
}

void quickSort(int arr[], int low, int high) {
    if (low < high) {
        int p = partition(arr, low, high);
        quickSort(arr, low, p);
        quickSort(arr, p + 1, high);
    }
}
```