# 1.
> Why the original `test_and_set` program does not satisfy bounded waiting?

因為它不計算一個行程連續進入 `critical_section` 的次數，也從而無法避免一個行程不斷霸佔 `critical_section`。

# 2.
> Please prove the correctness of the following extended `compare_and_swap` program in terms of mutual exclusion, progress, and bounded waiting.
>```c
>while (true) {
>    waiting[i] = true;
>    key = 1;
>    while (waiting[i] && key == 1) key = compare_and_swap(&lock, 0, 1);
>    waiting[i] = false;
>    /* critical section */
>    j = (i + 1) % n;
>    while ((j != i) && !waiting[j]) j = (j + 1) % n;
>    if (j == i)
>        lock = 0;
>    else
>        waiting[j] = false;
>}
>/* remainder section */
>```

> [!tip] 我的紙上已經有程式碼為什麼滿足這三個條件，在這裡要用說明的。

