# trace (系统调用的实现)
### step1:将trace加入MakeFile <br>
  ```$U/_trace\  ```<br>
### step2:实现trace这个系统调用，让他能跑的通，能够make qemu <br>
  - 首先在usys.pl给他一个入口 <br>
    ```entry("trace");```
  - 再在syscall.h中给trace一个系统调用号 <br>
    ```#define SYS_trace  22```
  - 再在user.h给一个trace函数的声明 <br>
    ```int trace(int);```
  - 再在sysproc.c中写个sys_trace()函数，仅让他返回0 <br>
    ```
    uint64
    sys_trace(void){
      return 0;
    }
    ``` 
  - 这样make qemu 就能编译成功了,初步完成雏形 <br>
### step3:sys_trace()的具体实现  
  ```
  uint64
  sys_trace(void)
  {
    struct proc *p = myproc();//获取进程
    int trace_mask;//用于接受trace的参数，例如trace 32中的32
    if(argint(0, &trace_mask) < 0){ //用argint()获取参数，第一个0是寄存器a0的意思
                                    //将a0寄存器中保存的参数写到trace_mask中
      return -1;
    }
    p->mask = trace_mask; //我们要再proc.h的proc结构体中定义一个mask
                        //使得mask成为进程的一个部分，用于syscall()里的输出判断
    return 0;
  }
  ```
### step4:修改syscall()
  * 在syscall.c中将```[SYS_trace]   sys_trace```加到```(*syscalls[])```中
  * ```//add a array of syscall to print its name for lab_trace```
    ```
    char *buf[] = {
      "fork", "exit", "wait", "pipe", "read", "kill", "exec", "fstat", "chdir",
      "dup", "getpid", "sbrk", "sleep", "uptime",
      "open", "write", "mknod", "unlink", "link", "mkdir", "close", "trace",
    };
    ```
  * 在syscall()函数中加入输出打印结果的代码
    ```
    void
    syscall(void)
    {
      int num;
      struct proc *p = myproc();

      num = p->trapframe->a7;
      if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
        p->trapframe->a0 = syscalls[num]();
        
        if((p->mask >> num) & 1){ //判断是否命中 
          //若命中则输出这种格式  3: syscall read -> 1023
          printf("%d: syscall %s -> %d\n", p->pid, buf[num - 1], p->trapframe->a0);
        }
        
      } else {
        printf("%d %s: unknown sys call %d\n",
                p->pid, p->name, num);
        p->trapframe->a0 = -1;
      }
    }
    ```
# sysinfo