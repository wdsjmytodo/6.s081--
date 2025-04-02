# backtrace 
### 这个lab让我们熟悉stack frame 这个内容
+ 实验流程:
  - make qemu , run bttest
  - 查看user/bttest.c, 它调用了sleep();
  - ->走到sys_sleep() in sysproc.c,因此我们要在sys_sleep()里调用backtrace()
  - Implement a backtrace() function in kernel/printf.c
+ 主要函数backtrace() 的实现:
  ```c
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
```c
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
## 2.test1()和test2()的部分
### ```要求: when the alarm handler is done, control returns to the instruction at which the user program was originally interrupted by the timer interrupt```
```在执行handler()之前通过在proc中新建一些变量来保存现场，然后执行完handler()后四个return函数要恢复现场```
```c
//保存:
void
preserve(void)
{
  struct proc *p = myproc();
  p->alarm_epc = p->trapframe->epc;
  p->alarm_ra = p->trapframe->ra;
  p->alarm_sp = p->trapframe->sp;
  p->alarm_gp = p->trapframe->gp;
  p->alarm_tp = p->trapframe->tp;
  p->alarm_t0 = p->trapframe->t0;
  p->alarm_t1 = p->trapframe->t1;
  p->alarm_t2 = p->trapframe->t2;
  p->alarm_s0 = p->trapframe->s0;
  p->alarm_s1 = p->trapframe->s1;
  p->alarm_a0 = p->trapframe->a0;
  p->alarm_a1 = p->trapframe->a1;
  p->alarm_a2 = p->trapframe->a2;
  p->alarm_a3 = p->trapframe->a3;
  p->alarm_a4 = p->trapframe->a4;
  p->alarm_a5 = p->trapframe->a5;
  p->alarm_a6 = p->trapframe->a6;
  p->alarm_a7 = p->trapframe->a7;
  p->alarm_s2 = p->trapframe->s2;
  p->alarm_s3 = p->trapframe->s3;
  p->alarm_s4 = p->trapframe->s4;
  p->alarm_s5 = p->trapframe->s5;
  p->alarm_s6 = p->trapframe->s6;
  p->alarm_s7 = p->trapframe->s7;
  p->alarm_s8 = p->trapframe->s8;
  p->alarm_s9 = p->trapframe->s9;
  p->alarm_s10 = p->trapframe->s10;
  p->alarm_s11 = p->trapframe->s11;
  p->alarm_t3 = p->trapframe->t3;
  p->alarm_t4 = p->trapframe->t4;
  p->alarm_t5 = p->trapframe->t5;
  p->alarm_t6 = p->trapframe->t6;
}

//恢复:
void
recover()
{
  struct proc *p = myproc();
  p->trapframe->epc = p->alarm_epc;
  p->trapframe->ra = p->alarm_ra;
  p->trapframe->sp = p->alarm_sp;
  p->trapframe->gp = p->alarm_gp;
  p->trapframe->tp = p->alarm_tp;
  p->trapframe->t0 = p->alarm_t0;
  p->trapframe->t1 = p->alarm_t1;
  p->trapframe->t2 = p->alarm_t2;
  p->trapframe->s0 = p->alarm_s0;
  p->trapframe->s1 = p->alarm_s1;
  p->trapframe->a0 = p->alarm_a0;
  p->trapframe->a1 = p->alarm_a1;
  p->trapframe->a2 = p->alarm_a2;
  p->trapframe->a3 = p->alarm_a3;
  p->trapframe->a4 = p->alarm_a4;
  p->trapframe->a5 = p->alarm_a5;
  p->trapframe->a6 = p->alarm_a6;
  p->trapframe->a7 = p->alarm_a7;
  p->trapframe->s2 = p->alarm_s2;
  p->trapframe->s3 = p->alarm_s3;
  p->trapframe->s4 = p->alarm_s4;
  p->trapframe->s5 = p->alarm_s5;
  p->trapframe->s6 = p->alarm_s6;
  p->trapframe->s7 = p->alarm_s7;
  p->trapframe->s8 = p->alarm_s8;
  p->trapframe->s9 = p->alarm_s9;
  p->trapframe->s10 = p->alarm_s10;
  p->trapframe->s11 = p->alarm_s11;
  p->trapframe->t3 = p->alarm_t3;
  p->trapframe->t4 = p->alarm_t4;
  p->trapframe->t5 = p->alarm_t5;
  p->trapframe->t6 = p->alarm_t6;
}


```

