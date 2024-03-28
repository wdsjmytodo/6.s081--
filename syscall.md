# trace (系统调用的实现)
### tips:
+ step1:添加trace这个syscall，让他能run
  - add trace to Makefile <br>
  <img src="https://s2.loli.net/2024/03/28/IMwvsfiYz7lg2Tc.png" alt="alt text" width="304" height="228">
  - add a prototype for the system call to user/user.h, a stub to user/usys.pl, and a syscall number to kernel/syscall.h. <br>
  ![image.png](https://s2.loli.net/2024/03/28/Pmbvq5jhYs7Mx3z.png)
+ step2:
