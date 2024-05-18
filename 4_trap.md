# backtrace 
### 这个lab让我们熟悉stack frame 这个内容
+ 实验流程:
  - make qemu , run bttest
  - 查看user/bttest.c, 它调用了sleep();
  - ->走到sys_sleep() in sysproc.c,因此我们要在sys_sleep()里调用backtrace()
  - Implement a backtrace() function in kernel/printf.c
+ 主要函数backtrace() 的实现:
  ```
      void
      backtrace(void)
      { 
            printf("bracktrace:\n");
            //fp: 0x0000003fffff9f88
            uint64 fp =  r_fp();
            uint64 *stack_frame = (uint64 *)fp;

            uint64 up = PGROUNDUP(fp);
            uint64 down = PGROUNDDOWN(fp);
            while (fp < up && fp > down)
            {
                  printf("%p\n", stack_frame[-1]);
                  fp = stack_frame[-2] ;
                  stack_frame = (uint64 *)fp;
            }
      }
  ```

# Alarm
## 1.test0()部分
### ```这个实验是通过时间中断的方式让程序转到handler```
+ user/alarmtest.c   add it to Makefile
+ sigalarm(ticks, fn)
  查看了alarmtest.c的test0(),先调用sigalarm(2, periodic())
  periodic()让count+1,并且输出alarm,然后调用sigreturn()
+ 添加系统调用sigalarm和sigreturn
+ add ticks, fn, ticks_count to struct proc in proc.h
  ticks_counts用于记录两个系统调用之间经过了多少个ticks
+ in trap.c 修改时间中断的部分
```
if(which_dev == 2){
    if(p->ticks > 0){
      p->ticks_count++;
      if(p->ticks_count > p->ticks){
        p->trapframe->epc = p->handler;
      }
    }
    yield();
  }
```