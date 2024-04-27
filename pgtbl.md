# 2021版labs
# 1.speed up system calls
+ 这个实验的目的是让我们对内存页有更加清晰的认识
  
- map the USYSCALL page just below TRAPFRAME page in proc_pagetable() in proc.c
  ```
  if(mappages(pagetable, USYSCALL, PGSIZE,
              (uint64)(p->usyscall), PTE_R | PTE_U) < 0){
    uvmunmap(pagetable, USYSCALL, 1, 0);
    uvmunmap(pagetable, TRAMPOLINE, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
  }
  ```
  - alloc and initialize usyscall page in allcoproc() in proc.c 
  ```
   if((p->usyscall = (struct usyscall *)kalloc()) == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }
   p->usyscall->pid = p->pid; //让结构体的pid直接等于进程的pid以实现加速
  ```
  - free the page in freeproc()
  ```
  //free usyscall page
  if(p->usyscall)
    kfree((void*)p->usyscall);
  p->usyscall = 0;
  ```