---
layout: notes
---

# Virtual Memory 

#### How to run different programs if they assume they are the only program running?

- All the `.o` files share address space, and the linker would merge them together into a continuous chunk 



#### Non-Solution 

- Programs take turns to run 
  - BAD: this could be slow
- Linker relocate one last time 



#### Better Non-Solution

- Continuously map programs to different places (process id => base + offset)

**Problems:** 

- Gowing of processes are hard 
- Entire program in memory wasteful (only partial residence is needed) 
- External fragmentation 



#### Solution - Paging 

- Use more fine grained, fix size "Page" 

**Benefits:** 

- Can grow dynamically 
- no more external fragmentation 
- partial memory resident possible 



### How to do page to frame translation?

- Linear address part of it being the Page #
- Use a Page Table with Page # as index, and entry as Frame #
- Use a Page Tabel Base Register for different processes

