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

# 2.print a pagetble
+ 这个实验加深我们对于xv6的三级页表理解
  - Insert if(p->pid==1) vmprint(p->pagetable, 0) in exec.c just before the return argc, to print the first process's page table.//这里我改了一下，将页表的级数也传进来以区别是第几级页表。加这段代码使得exec能够运行vmprint()。
  ```
  if(p->pid==1)
   vmprint(p->pagetable, 0);
  ```
  - implement of vmprint() in vm.c 。(可以参照freewalk函数对页表的递归调用)
  ```
  void
  vmprint(pagetable_t pagetable, int depth)
  {
    //print first line
    if(depth == 0)
      printf("page table %p\n", pagetable);
    //..0: pte 0x0000000021fda801 pa 0x0000000087f6a000
    // there are 2^9 = 512 PTEs in a page table.
    for(int i = 0; i < 512; i++){
      pte_t pte = pagetable[i];
      if(pte & PTE_V ){
        // this PTE points to a lower-level page table.
        if(depth == 0){
          printf("..%d: pte %p pa %p\n", i, pte, PTE2PA(pte));
        }
        if(depth == 1){
          printf(".. ..%d: pte %p pa %p\n", i, pte, PTE2PA(pte));
        }
        if(depth == 2){
          printf(".. .. ..%d: pte %p pa %p\n", i, pte, PTE2PA(pte));
        }
        if(depth > 2){//if depth > 2 ,return
          return;
        }
        uint64 child = PTE2PA(pte);
        vmprint((pagetable_t)child, depth + 1);
      }
    }
  }
  ```
  - 记得在defs.h中声明vmprint()
  ```
  void            vmprint(pagetable_t, int);
  ```