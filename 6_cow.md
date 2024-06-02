# COW-fork()
+ 1. Modify uvmcopy()   Clear PTE_W 
+ 2. Modify usertrap() PTE_W set
+ 3. Set a page's reference count to one when kalloc() allocates it.kfree() should only place a page back on the free list if its reference count is zero. 
+ 