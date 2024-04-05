# trace (系统调用的实现)
### step1:将trace加入MakeFile <br>
  $U/_trace\  <br>
### step2:实现trace这个系统调用，让他能跑的通，能够make qemu <br>
  - 首先在usys.pl给他一个入口 <br>
    entry("trace");
  - 再在syscall.h中给trace一个系统调用号 <br>
    #define SYS_trace  22
  - 再在user.h给一个trace函数的声明 <br>
    int trace(int);
  - 再在sysproc.c中写个sys_trace()函数，仅让他返回0 <br>
  这样make qemu 就能编译成功了 <br>
### step3:sys_trace()的具体实现