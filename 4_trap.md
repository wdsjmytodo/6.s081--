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