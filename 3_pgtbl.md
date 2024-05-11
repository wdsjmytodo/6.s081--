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

# 3.detecting which page has been accessed
+ 理解这个实验叫我们干什么，因此要参考一下pgtbltest()函数
+ for me 从这个lab,可知
  - sysproc.c中的函数都是系统调用函数，是给user 用来调用的，因此像sys_pgaccess()用argaddr,argint 接受的参数都是属于用户的，因此使用copyout()时，这些参数当作destination，从而将内核的信息copy到这些参数进而上交给用户
  - vm_pgaccess要参照wlakaddr(pagetable, va)和walk(pagetable, va, 0)，这种函数是把va传进去，然后获得pte【这个主要是walk()的作用】，然后验证标志位，从而达到我们想要的目的

### sys_pgaccess()
```
sys_pgaccess(void)
{
  // lab pgtbl: your code here.

  
  //int pgaccess(void *base, int len, void *mask);
  //接收参数，并且这些参数可以视作都是用户的

  //base是起始页地址,base、len、mask都是用户的
  uint64 base;
  int len;
  int mask;
  if(argaddr(0, &base) < 0)
    return -1;
  if(argint(1, &len) < 0)
    return -1;
  if(argint(2, &mask) < 0)
    return -1;
  //设置len
  if( len < 0 || len > 32){
    return -1;
  }
  struct proc *p = myproc();

  //用result来接受内核中的访问结果，result应该为32位，因为pgtbltest测试中len=32
  //for循环通过vm_pgaccess()函数检验page的PTE_A位来确定abit的值，然后result移位
  //result的第几位是1代表第几个page被访问了
  int result = 0;
  for(int i = 0; i < len; i++){
    int va = base + i * PGSIZE;

    //abit = 0 / 1
    //vm_pgaccess参照walkaddr()
    int abit = vm_pgaccess(p->pagetable, va);

    //result 移位
    result = result | abit << i;
  }

  //将result copyout to mask
  if(copyout(p->pagetable, mask, (char*)&result, sizeof(result)) < 0){
    return -1;
  }
  return 0;
}
```  

### vm_pgaccess() implement in vm.c
```
//return 0 if pte's PTE_A = 0
//return 1 if pte's PTE_A = 1, as the same time, reset PTE_A to 0
int
vm_pgaccess(pagetable_t pagetable, uint64 va)
{
  pte_t *pte;

  if(va >= MAXVA)
    return 0;

  pte = walk(pagetable, va, 0);
  if(pte == 0)
    return 0;
  
  if((*pte & PTE_A) != 0){
    *pte = *pte & (~PTE_A);
    return 1; 
  }

  return 0;
}
```