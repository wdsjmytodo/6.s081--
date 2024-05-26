# Eliminate allocation from sbrk()
```在sys_sbrk()中: ```
```
// if(growproc(n) < 0)  //将这两行注释掉
  //   return -1;
  myproc()->sz += n; //只让sz的大小提升，不实际分配空间
```
# Lazy allocation
``` 在usertrap()中添加:```
  ```
    else if(r_scause() == 13 || r_scause() == 15){//用于判断中断来自lazy-alloc的异常情况
    uint64 va = r_stval();
    if(is_lazy_alloc_va(va)){//用两个函数来实现这里的判断
      if(lazy_alloc(va) < 0){
        printf("lazy_alloc failed\n");
        myproc()->killed = 1;
      }
    }
  } 
  ```
  ``` in proc.c ,完成is_lazy_alloc_va(va)和lazy_alloc(va)```
  ```
  int
  is_lazy_alloc_va(uint64 va){
    if(va > myproc()->sz){
      return 0;
    }
    return 1;
  }

  int 
  lazy_alloc(uint64 va){
    va = PGROUNDDOWN(va);
    struct proc* p = myproc();
    char* mem = kalloc();
    if(mem == 0){
      return -1;
    }
    memset(mem, 0, PGSIZE);
    if(mappages(p->pagetable, va, PGSIZE, (uint64)mem, PTE_W|PTE_X|PTE_R|PTE_U) != 0){
      kfree(mem);
      return -1;
    }  
    return 0;
  }
  ```

# Lazytests and Usertests
+ 1.Handle negative sbrk() arguments
      ```

      ```
+ 2.Kill a process if it page-faults on a virtual memory address higher than any allocated with sbrk()
  ```

  ```
+ 3.Handle the parent-to-child memory copy in fork() correctly.P

+ 4.
+ 5.Handle out-of-memory correctly: if kalloc() fails in the page fault handler, kill the current process.
  ```这个在trap.c的判断里就做了的```
  ```
  if(lazy_alloc(va) < 0){
        printf("lazy_alloc failed\n");
        myproc()->killed = 1;
      }
  ```
  
+ 6.Handle faults on the invalid page below the user stack.
  ```当va用到了stack下面的guard page 的地址，则杀死进程```
  ```
  
  ```