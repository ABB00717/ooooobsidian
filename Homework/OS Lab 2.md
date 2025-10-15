作業要求為完成以下排程器：
1. 循環法
	- timertick
2. 優先度
	- prio
3. 先到先來
4. 最短工作優先
	- burst

# 循環法（RR）
預設就已經有循環法了。我們只要實現 -timertick 就好，這也非常簡單。機器的所有狀態都存
在 machine/stats.h ，當然也會有我們要的 TimerTicks 。先做點小改動，因為原本的 TimerTicks 是 `const`，我們不該直接把它改掉。

觀察哪裡有用到 `TimerTicks`。

```bash
# 結果有經過刪減
$ grep -RI TimerTicks
machine/stats.h:const int TimerTicks = 	 100;  	
machine/timer.cc:       int delay = TimerTicks;
machine/timer.cc:	     delay = 1 + (RandomNumber() % (TimerTicks * 2));
```

而我們填入參數的地方在 `threads/kernel.cc`，所以想辦法把 `threads/kernel.cc` 的數值傳遞到 `machine/timer.cc` 就好了。看能不能在初始化 `Timer` 的時候傳遞。

```bash
$ grep -RI --exclude="*timer.*" Timer\(
threads/alarm.cc:    timer = new Timer(doRandom, this);
```

唯一只有在 `alarm.cc` 有初始 `Timer`。

```bash
$ grep -RI --exclude="*alarm.*" Alarm\(
threads/kernel.cc:    alarm = new Alarm(randomSlice);	// start up time slicing
```

只有在 `kernel.cc` 有初始化 `Alarm` 。剛好，我們可以透過 `Alarm` 和 `Timer` 傳遞值。

先從 argc 裡讀取 -timertick 。

```c
// threads/kernel.h
/*
class ThreadedKernel {
...
  private:
...
*/
    int timerTicks;
/*
};
*/
```

```c
// threads/kernel.cc
/*
ThreadedKernel::ThreadedKernel(int argc, char **argv) {
...
	bool cus_timer = false;
	for (int i = 1; i < argc; i++) {
...
*/
		} else if (strcmp(argv[i], "-timertick") == 0) {
			if (!(i + 1 < argc)) {
				cout << "Partial usage: nachos [-timertick TimerTicks]\n";
				continue;
			}
		
		// cout << "TimerTicks: " << atoi(argv[i + 1]) << endl;
		timerTicks = atoi(argv[i + 1]);
		cus_timer = true;
		i++;
		}	
	}
	
	if (!cus_timer)
		timerTicks = TimerTicks;
}
/*
...
void ThreadedKernel::Initialize() {
...
*/
    alarm = new Alarm(randomSlice, timerTicks);   // start up time slicing
/*
...
}
*/
```

修改 `Alarm` 的建構子

```c
// threads/alarm.h
/*
class Alarm : public CallBackObj {
  public:
...
*/
    Alarm(bool doRandomYield, int timerTicks);
/*
...
};
*/
```

```c
/// threads/alarm.cc
Alarm::Alarm(bool doRandom, int timerTicks) { timer = new Timer(doRandom, timerTicks, this); }
```

修改 `Timer` 的建構子

```c
// machine/timer.h
/*
class Timer : public CallBackObj {
...
  public:
*/
    Timer(bool doRandom, int timerTicks, CallBackObj *toCall);
/*
...
   private:
*/
	int timerDelay
/*
...
};
*/
```

```c
// machine/timer.cc
/*
Timer::Timer(bool doRandom, int timerTicks, CallBackObj *toCall)
{
...
*/
    timerDelay = timerTicks;
/*
...
}
...
void Timer::SetInterrupt() {
*/
    if (!disable) {
       int delay = timerDelay;
    
       if (randomize) {
	     delay = 1 + (RandomNumber() % (timerDelay * 2));
        }
/*
       // schedule the next timer device interrupt
       kernel->interrupt->Schedule(this, delay, TimerInt);
    }
}
*/
```

## 測資測試

```
$ ./threads/nachos -sche RR
A: 2
A: 1
A: 0
B: 9
C: 3
C: 2
C: 1
C: 0
B: 8
B: 7
B: 6
B: 5
B: 4
B: 3
B: 2
B: 1
B: 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 2600, idle 130, system 2470, user 0
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

```
$ ./userprog/nachos -e ./test/test1 -e ./test/test2 -e ./test/test3 -sche RR
Total threads number is 3
Thread ./test/test1 is executing.
Thread ./test/test2 is executing.
Thread ./test/test3 is executing.
Print integer:9
Print integer:8
Print integer:2020
Print integer:2021
Print integer:2022
Print integer:2023
Print integer:2024
Print integer:10000
Print integer:10001
Print integer:10002
Print integer:10003
Print integer:10004
Print integer:7
Print integer:6
Print integer:5
Print integer:4
Print integer:3
Print integer:2
Print integer:2025
Print integer:2026
Print integer:2027
Print integer:2028
Print integer:2029
Print integer:10005
Print integer:10006
Print integer:10007
Print integer:10008
Print integer:10009
Print integer:1
return value:0
Print integer:2030
return value:0
Print integer:10010
return value:0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 800, idle 65, system 140, user 595
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

```
$ ./threads/nachos -sche RR -timertick 20
A: 2
A: 1
B: 9
A: 0
B: 8
C: 3
B: 7
C: 2
C: 1
B: 6
C: 0
B: 5
B: 4
B: 3
B: 2
B: 1
B: 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 4803, idle 93, system 4710, user 0
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

```
$ ./userprog/nachos -e ./test/test1 -e ./test/test2 -e ./test/test3 -sche RR -timertick 60
Total threads number is 3
Thread ./test/test1 is executing.
Thread ./test/test2 is executing.
Thread ./test/test3 is executing.
Print integer:2020
Print integer:2021
Print integer:10000
Print integer:10001
Print integer:9
Print integer:8
Print integer:7
Print integer:2022
Print integer:2023
Print integer:2024
Print integer:10002
Print integer:10003
Print integer:10004
Print integer:6
Print integer:5
Print integer:4
Print integer:2025
Print integer:2026
Print integer:2027
Print integer:10005
Print integer:10006
Print integer:10007
Print integer:3
Print integer:2
Print integer:1
Print integer:2028
Print integer:2029
Print integer:2030
Print integer:10008
Print integer:10009
Print integer:10010
return value:0
return value:0
return value:0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 840, idle 45, system 200, user 595
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

# 優先度（PRIORITY）
專案已經實做部份PRIORITY了，不過是搶佔式的，但作業要求不要。因此把PRIORITY從 YieldOnReturn的判斷刪除。


```c
// threads/alarm.cc
/*
void Alarm::CallBack() {
...
	if (status == IdleMode) { // is it time to quit?
...
*/
	} else { // there's someone to preempt
		if (kernel->scheduler->getSchedulerType() == RR) {
			interrupt->YieldOnReturn();
		}
/*
	 }
}
*/
```

接著再完成作業要求的 -prio 。在 `UserProgKernel` 新增陣列 `priority` 儲存優先度。

```c
// userprog/userkernel.h
/*
class UserProgKernel : public ThreadedKernel {
...
  private:
...
*/
	int priority[10];
/*
...
};
...
*/
```

實作從 argv 裡讀取優先度。

```c
// userprog/userkernel.cc
/*
UserProgKernel::UserProgKernel(int argc, char **argv)
: ThreadedKernel(argc, argv) {
	debugUserProg = FALSE;
	execfileNum = 0;
	for (int i = 1; i < argc; i++) {
		if (strcmp(argv[i], "-s") == 0) {
			debugUserProg = TRUE;
*/
		} else if (strcmp(argv[i], "-e") == 0) {
			execfile[++execfileNum] = argv[i + 1];
			i++;
		 if (strcmp(argv[i + 1], "-prio") == 0) {
			 i += 2;
			 ASSERT(i < argc);
			 priority[execfileNum] = atoi(argv[i]);
		 }
/*
...
	 }
...
*/
```

補全 ReadyToRun ，在這裡就順便把其他的也都加上去。

```c
// threads/scheduler.cc
/*
void Scheduler::ReadyToRun(Thread *thread) {
...
	if (schedulerType == RR || schedulerType == FIFO) {
		readyList->Append(thread);
*/
	} else if (schedulerType == SJF || schedulerType == Priority) {
		((SortedList<Thread *> *)readyList)->Insert(thread);
/* 
	} else {
		readyList->Append(thread);
	}
}
```

## 測資測試
```
$ ./threads/nachos -sche PRIORITY
C: 3
C: 2
C: 1
C: 0
A: 2
A: 1
A: 0
B: 9
B: 8
B: 7
B: 6
B: 5
B: 4
B: 3
B: 2
B: 1
B: 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 2400, idle 40, system 2360, user 0
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

```
$ ./userprog/nachos -e ./test/test1 -prio 10 -e ./test/test2 -prio 20 -e ./test/test3 -prio 3 -sche PRIORITY
Total threads number is 3
Thread ./test/test1 is executing.
Thread ./test/test2 is executing.
Thread ./test/test3 is executing.
Print integer:10000
Print integer:10001
Print integer:10002
Print integer:10003
Print integer:10004
Print integer:10005
Print integer:10006
Print integer:10007
Print integer:10008
Print integer:10009
Print integer:10010
return value:0
Print integer:9
Print integer:8
Print integer:7
Print integer:6
Print integer:5
Print integer:4
Print integer:3
Print integer:2
Print integer:1
return value:0
Print integer:2020
Print integer:2021
Print integer:2022
Print integer:2023
Print integer:2024
Print integer:2025
Print integer:2026
Print integer:2027
Print integer:2028
Print integer:2029
Print integer:2030
return value:0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 700, idle 35, system 70, user 595
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

# 先到先來（FCFS)
先到先來，所以就無腦一直放進一個 List 的末端就好了。

```c
// threads/scheduler.cc
/*
Scheduler::Scheduler(SchedulerType type) {
	schedulerType = type;
	switch (schedulerType) {
...
*/
		case FIFO:
			readyList = new List<Thread *>;
			break;
/*
	}
...
}
*/
```

```c
// threads/scheduler.cc
/*
void Scheduler::ReadyToRun(Thread *thread) {
...
*/
	if (schedulerType == RR || schedulerType == FIFO) {
		readyList->Append(thread);
/*
	} else if (schedulerType == SJF || schedulerType == Priority) {
		((SortedList<Thread *> *)readyList)->Insert(thread);
	} else {
		readyList->Append(thread);
	}
}
*/
```

## 測資測試
```
$ ./threads/nachos -sche FCFS
A: 2
A: 1
A: 0
B: 9
B: 8
B: 7
B: 6
B: 5
B: 4
B: 3
B: 2
B: 1
B: 0
C: 3
C: 2
C: 1
C: 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 2500, idle 150, system 2350, user 0
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

```
$ ./userprog/nachos -e ./test/test1 -e ./test/test2 -e ./test/test3 -sche FCFS
Total threads number is 3
Thread ./test/test1 is executing.
Thread ./test/test2 is executing.
Thread ./test/test3 is executing.
Print integer:9
Print integer:8
Print integer:7
Print integer:6
Print integer:5
Print integer:4
Print integer:3
Print integer:2
Print integer:1
return value:0
Print integer:2020
Print integer:2021
Print integer:2022
Print integer:2023
Print integer:2024
Print integer:2025
Print integer:2026
Print integer:2027
Print integer:2028
Print integer:2029
Print integer:2030
return value:0
Print integer:10000
Print integer:10001
Print integer:10002
Print integer:10003
Print integer:10004
Print integer:10005
Print integer:10006
Print integer:10007
Print integer:10008
Print integer:10009
Print integer:10010
return value:0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 700, idle 35, system 70, user 595
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

# 最短工作優先（SJF）
要維護一個 SortedList ，以最短工作時間排到最長。

```c
// threads/scheduler.cc
/*
...
*/
// SJFCompare function
int SJFCompare(Thread *a, Thread *b) {
	if (a->getBurstTime() == b->getBurstTime()) return 0;
	return a->getBurstTime() > b->getBurstTime() ? 1 : -1;
}
/*
...
Scheduler::Scheduler(SchedulerType type) {
	schedulerType = type;
	switch (schedulerType) {
*/
		case SJF:
			readyList = new SortedList<Thread *>(SJFCompare);
			break;
/*
...
	}
	
	toBeDestroyed = NULL;
}
...
void Scheduler::ReadyToRun(Thread *thread) {
...
	if (schedulerType == RR || schedulerType == FIFO) {
		readyList->Append(thread);
*/
	} else if (schedulerType == SJF || schedulerType == Priority) {
		((SortedList<Thread *> *)readyList)->Insert(thread);
/*
	} else {
		 readyList->Append(thread);
	 }
}
*/
```

接著再完成作業要求的 -burst 。在 UserProgKernel 新增陣列 burst 儲存每個執行緒的工作時間。

```c
// userprog/userprog.h
/*
class UserProgKernel : public ThreadedKernel {
...
  private:
...
*/
	int burst[10];
/*
...
};
...
*/
```

實作從 argv 裡讀取工作時間。

```c
// userprog/userkernel.cc
/*
UserProgKernel::UserProgKernel(int argc, char **argv)
	: ThreadedKernel(argc, argv) {
	debugUserProg = FALSE;
	execfileNum = 0;
	for (int i = 1; i < argc; i++) {
		if (strcmp(argv[i], "-s") == 0) {
			debugUserProg = TRUE;
			if (strcmp(argv[i + 1], "-prio") == 0) {
				i += 2;
				ASSERT(i < argc);
				priority[execfileNum] = atoi(argv[i]);
*/
			 } else if (strcmp(argv[i + 1], "-burst") == 0) {
				 i += 2;
				 ASSERT(i < argc);
				 burst[execfileNum] = atoi(argv[i]);
			 }
/*
...
		}
...
```

## 測資測試
```
abb00717@ABB00717:~/Projects/NachOS/code$ ./threads/nachos -sche SJF
A: 2
A: 1
A: 0
C: 3
C: 2
C: 1
C: 0
B: 9
B: 8
B: 7
B: 6
B: 5
B: 4
B: 3
B: 2
B: 1
B: 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 2400, idle 50, system 2350, user 0
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```

```
$ ./userprog/nachos -e ./test/test1 -burst 8 -e ./test/test2 -burst 2 -e ./test/test3 -burst 5 -sche SJF
Total threads number is 3
Thread ./test/test1 is executing.
Thread ./test/test2 is executing.
Thread ./test/test3 is executing.
Print integer:2020
Print integer:2021
Print integer:2022
Print integer:2023
Print integer:2024
Print integer:2025
Print integer:2026
Print integer:2027
Print integer:2028
Print integer:2029
Print integer:2030
return value:0
Print integer:10000
Print integer:10001
Print integer:10002
Print integer:10003
Print integer:10004
Print integer:10005
Print integer:10006
Print integer:10007
Print integer:10008
Print integer:10009
Print integer:10010
return value:0
Print integer:9
Print integer:8
Print integer:7
Print integer:6
Print integer:5
Print integer:4
Print integer:3
Print integer:2
Print integer:1
return value:0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!

Ticks: total 700, idle 35, system 70, user 595
Disk I/O: reads 0, writes 0
Console I/O: reads 0, writes 0
Paging: faults 0
Network I/O: packets received 0, sent 0
```