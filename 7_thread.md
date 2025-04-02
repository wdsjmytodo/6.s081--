# 1.Uthread: switching between threads
```Requirement:you will design the context switch mechanism for a user-level threading system```
+ question: in `uthread.c` and `uthread_switch.S`:The threading package is missing some of the code to create a thread and to switch between threads.
+ solution: you will need to add code to `thread_create()` and `thread_schedule()` in `user/uthread.c`, and `thread_switch` in `user/uthread_switch.S`. 
