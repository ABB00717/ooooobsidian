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

## Mutual
行程 $P_i$ 要進入 `critical section`，必須跳出 `while` 迴圈。跳出 `while` 有兩種辦法。

- `key == 0`：
  `compare_and_swap` 在 `lock == 0` 時會讓 `key = 0`。因為它是原子操作，只有一個行程能成功。
- `waiting[i] == false`：
  只能由別的行程在 `exit section` 完成。

因此任意時間點只有一個行程能進入 `critical section`。
## Progress
- `lock == 0`：
  任何想進去的行程一定有一個可以進去（參見 Mutual）
- `lock == 1`：
  離開的行程在 `exit section`會：
	- 找到下個正在等待的 $P_j$，`waiting[j] = false`，讓 $P_j$ 直接進入 `critical section`
	- 沒有其他行程在等待，`lock = 0`。

系統總可以從想進入的行程中挑一個進入 `critical section`
## Bounded Waiting
行程 $P_i$ 在 `exit section` 會從 `j = (i +1) % n` 開始循環尋找下個正在等待的行程。因此行程的等待上限是 `n-1`。
## a.
> Please write a program for producer-consumer (i.e., bounded buffer) problem with buffer size n by using a counter.

全域變數 `count = 0, in = 0, out = 0`

```c
// Producer Process
while (true) {
	// produce next_produced ..
	while (count == n);
	
	buffer[in] = next_produced;
	in = (in + 1) % n;
    count++;
}
```

```c
// Consumer Process
while (true) {
    while (count == 0);

    next_consumed = buffer[out];
    out = (out + 1) % n;
    count--;
}
```

# b.
> What is the race condition and does the race condition occur in the solution of (a) you wrote? Why?

會有競賽條件，因為生產者和消費者能夠同時操作 `count`，而他們又不是原子操作，會有一方用過時的 `count` 覆蓋掉另一方的操作結果，造成 Lost Update。

# 3.
> For the following two threads shared x and flag
> ![[Pasted image 20251016152014.png]]
> How to apply Memory barrier into both threads to ensure the output must be 100?
> Please write the program.

```c
// Thread 1
while (!flag);
memory_barrier();
print x;
```

```c
// Thread 2
x = 100;
memory_barrier();
flag = true;
```

# 4.
> Why we need hardware solution for synchronization? Please point out two possible drawbacks of software solution for critical section problem (such as Peterson’s Solution)?

一個高階語言指令底層是由許多條組合語言指令組成的，現代 CPU 以及編譯器會為了優化而亂序排序這些指令。

首先，若在 `entry section` 的某些關鍵函式在執行時被中斷切換到別的行程，可能會讓多個行程同時進入 `critical section`。其次，倘若為了優化，把 `entry section` 關鍵的驗證與 `critical section` 混合在一起，就又會產生

 因此會需要原子操作，避免上述情況發生。