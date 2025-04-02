# COW-fork()
+ Modify uvmcopy()   ,Clear PTE_W 
  ```不alloc，并将其设置为可读的```
+ Modify usertrap() to recognize page faults. ```r_scause()```When a page-fault occurs on a COW page, allocate a new page with kalloc()```char *mem```, copy the old page to the new page```memmove()```, and install the new page in the PTE with PTE_W set```mappages()```.
  
```

```
+ Set a page's reference count to one when kalloc() allocates it.kfree() should only place a page back on the free list if its reference count is zero. It's OK to to keep these counts in a fixed-size array of integers.
+ Modify copyout()