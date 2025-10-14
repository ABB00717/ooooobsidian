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

```c

```

只有在 `Timer` 裡面有用到，想辦法

```c
// machine/stats.h
...
extern int TimerTicks; // (average) time between timer interrupts
...
```

```c
// machine/stats.cc
...
int TimerTicks = 100;
...
```

再從 argc 裡讀取 -timertick 。

```c
// threads/kernel.cc
/*
ThreadedKernel::ThreadedKernel(int argc, char **argv) {
...
	for (int i = 1; i < argc; i++) {
...
*/
		} else if (strcmp(argv[i], "-timertick") == 0) {
			if (!(i + 1 < argc)) {
			cout << "Partial usage: nachos [-timertick TimerTicks]\n";
			continue;
		}
		// cout << "TimerTicks: " << atoi(argv[i + 1]) << endl;
		TimerTicks = atoi(argv[i + 1]);
		i++;
		}
/*
	}
}
```

## 測資測試


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

```c
// userprog/userprog.h
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