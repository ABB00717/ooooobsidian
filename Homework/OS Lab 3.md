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

## a.
> Please write a program for producer-consumer (i.e., bounded buffer) problem with buffer size n by using a counter.



# b.
> What is the race condition and does the race condition occur in the solution of (a) you wrote? Why?

# 3.
> For the following two threads shared x and flag
> ![[Pasted image 20251016152014.png]]
> How to apply Memory barrier into both threads to ensure the output must be 100?
> Please write the program.

# 4.
> Why we need hardware solution for synchronization? Please point out two possible drawbacks of software solution for critical section problem (such as Peterson’s Solution)?

一個高階語言指令底層是由許多條組合語言指令組成的，現代 CPU 以及編譯器會為了優化而亂序排序這些指令。

首先，若在 `entry section` 的某些關鍵函式在執行時被中斷切換到別的行程，可能會讓多個行程同時進入 `critical section`。其次，倘若為了優化，把 `entry section` 關鍵的驗證與 `critical section` 混合在一起，就又會產生

 因此會需要原子操作，避免上述情況發生。