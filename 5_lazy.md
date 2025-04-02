# Eliminate allocation from sbrk()
```在sys_sbrk()中: ```
```c
// if(growproc(n) < 0)  //将这两行注释掉
  //   return -1;
  myproc()->sz += n; //只让sz的大小提升，不实际分配空间
```
# Lazy allocation
``` 在usertrap()中添加:```
  ```c
  else if(r_scause() == 13 || r_scause() == 15){//用于判断中断来自lazy-alloc的异常情况
    uint64 va = r_stval();
    if(is_lazy_alloc_va(va)){//用两个函数来实现这里的判断
      if(lazy_alloc(va) < 0){
        printf("lazy_alloc failed\n");
        myproc()->killed = 1;
      }
    }
  }else {//其余不合法的地址的情况
    printf("usertrap() : try to access an illegal address\n");
    myproc()->killed = 1;
  } 
  ```
  ``` in proc.c ,完成is_lazy_alloc_va(va)和lazy_alloc(va)```
  ```c
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
  ``` echo hi 可输出hi就成功了```

# Lazytests and Usertests
+ 1.Handle negative sbrk() arguments.
  ```in sys_sbrk() ```
  ```c
  if(n > 0){
    p->sz += n;
  }else {
    if(p->sz + n < 0){
      return -1;
    }else{ //参照原来的growproc()函数里面的做法
      p->sz = uvmdealloc(p->pagetable, p->sz, p->sz + n);
    }
  }
  ```
+ 2.Kill a process if it page-faults on a virtual memory address higher than any allocated with sbrk().
  ```已经在lazy allocation中做好了```
+ 3.Handle the parent-to-child memory copy in fork() correctly.
  ```fork() 会调用uvmcopy(),在uvmcopy()中把相关的panic直接跳过 ```
  ```c
  if((pte = walk(old, i, 0)) == 0)
      // panic("uvmcopy: pte should exist");
      continue;
  if((*pte & PTE_V) == 0)
    // panic("uvmcopy: page not present");
    continue;
  ```

+ 4.Handle the case in which a process passes a valid address from sbrk() to a system call such as read or write, but the memory for that address has not yet been allocated.
  ```当read()或write()时，walkaddr函数可能会访问到懒分配的地址,因此要在walkaddr中对va做出判断，如果是懒分配的地址,则要给它分配物理空间再递归wlakaddr() ```
  ``` 将几个判断合在一起```
  ```c
  if(pte == 0 || (*pte & PTE_V) == 0 || (*pte & PTE_U) == 0){
    if(is_lazy_alloc_va(va)){
      if(lazy_alloc(va) < 0){
        return 0;
      }
      return walkaddr(pagetable, va);
    }
    return 0;
  }
  ```
+ 5.Handle out-of-memory correctly: if kalloc() fails in the page fault handler, kill the current process.
  ```这个在trap.c的判断里就做了的```
  ```c
  if(lazy_alloc(va) < 0){
        printf("lazy_alloc failed\n");
        myproc()->killed = 1;
      }
  ```
  
+ 6.Handle faults on the invalid page below the user stack.
  ```当va用到了stack下面的guard page 的地址，则杀死进程```
  ```c
  if(va < PGROUNDDOWN(myproc()->trapframe->sp) && va >= PGROUNDDOWN(myproc()->trapframe->sp) - PGSIZE){
    return 0;
  }
  ```