---
layout: post
author: Ricky
title: "Virtual Memory"
subtitle: 'OS notes'
tags: 
    - notes
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



### Double access of memory?

For example: 

```asm
movl (%esi), %eax 
```

1. Get PTE of %esi 
2. Get the %esi data 



### What happen if you have large memory address space, like 4GB?

- You will have MBs of page table , EACH PROCESS 

**Solution 1: Page Table Length Register **

- Restrict some of the pages only 



**Solution 2: Multi-level Paging** 

Why this: 

- Page tables mappings are a sparse list of dense lists:
  - many holes => sparse 
  - sequential access so that pages are together (spatial locality) => dense 



How is it space saving 

- Some of the PDEs can just be NULL 
