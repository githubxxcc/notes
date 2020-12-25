# Daily Notes 





## OS 

##### Segment descriptor: 

- 64 bits that describe: base address / limit / privilege level / ... of a segment 
- The calculated virtual linear address will then be used to access the virtual memory page structures 

![image-20200912144051157](/Users/chenxu/Documents/notes/imgs/image-20200912144051157.png)

##### Segment selector: 

- 16 bits of information that is stored in the segment register (%CS, %DS, ..) that tells you where to find the segment descriptor 
- RPL: uses as current p

![image-20200912144212055](/Users/chenxu/Documents/notes/imgs/image-20200912144212055.png)







P1 Design Doc 

Files structure: 

```
/kern 
	inc/ 
		int_handle.h 					// interrupt handler declarations
	
	int_handle.c 						// C implementation of the handlers
	int_handle_asm.S				// Assembly wrapper of the handlers 
	
	console.c 							// Handle specific functions implementation

	
```

Functions: 

```
FILE int_handle.c
	FUNC handler_install
		- Get the IDT table 
    - Install trap gate for each device 
    
  FUNC timer_init
  	- Initialize the timer by configuring through IO ports
  	- Install handler 
  	
  FUNC timer_handle
  	- run the callback 
  		**!**: How to store the callback 
  	- reply to the hardware 
  	
 
	 
FILE video.c
	FUNC putbyte:
		- get_cursor
		- Do:
			1. \n => switch line/scroll 
			2. \r => change cursor 
			3. \b => find previous char, no cross line 

	FUNC putbytes
		- use putbyte() 
		- Assume the str is valid (ASSUME) 
		
	FUN draw_char: 
		- Need to check color code / positions 
		- No check on char (ASSUME)

	FUNC set_term_color
		- Need to check color code
		
	FUNC clear_console:
		- hidden cursor stays hidden 
		- could not do a memset since the color needs to be correct
		
	FUNC scroll_up:
		- Need to copy the color bytes as well
		- How about the color of the last line (use the term_color) (DESIGN)
	
	VAR cur_pos:
		- row
		- col
	VAR color
	
	
FILE keyboard.c
	VAR head: position to push the data 
	VAR tail: position to pop the data
	FUNC push_byte: 
		- if the next pos is not the tail, then
		- add a byte to the head of the ring buffer 
		- advance the head 
		
	FUNC readchar:
		- read the scancode from buffer
		- use the HASDATA | MAKE scancode to get the result
		
		
	
	



```



Game Design 

```
FUNC kernel_main:
	- Handle interrupt handlers 
	- start_game()
	


FUNC start_game:
	- print introduction (intro())
	- setup the 
```

```
COMPONENT TILE_STATE 
	STATEs
		- cur_color: 		current color code
		- cur_ch:				current char content
		- content: 			underlying state (EMPTY, END_POINT)
		- orig_color: 	original color
		- orig_ch:			original char content 
	
	ACTIONs
		- init(x):
			given an original char, intilaize the tile
			
		- draw(color, ch):
			draw a specific color and character 
		
		- revert:
			back to its original display 
		
		- can_draw:
			test if this tile can be connected to some color pipes. 
			Only an not connected empty tile can be drawn. 
			
		- connected():
			test if this tile has been part of connected pipe. 
			A connected pipe's orig color/ch differs from the current ones

			
	
	

COMPONENT  BOARD
	STATEs
  	- tile_states: 	array of tile_state
  	- op_stack: 		stack of operations started from current drawing 
  	- num_endpts: 	number of end point pairs remaining 
  	- running: 			if the game is running  
  	- draw_color:		the current color being drawn, -1 for not drawing

  
  ACTIONs
  	- handle:
  		an entry point to handle the scancodes from the keyboard and dispatch
  		corresponding events
  	
  	- start_draw()
  		enter the drawing mode. 
  		If the current tile is not a valid startng point, it has no effect. 
  		Only an end point not yet connected can start the drawing 
  	
  	- move(dx, dy):
  		move the cursor if not in the drawing mode 
  		move the cursor and draw if in the crawing mode and can be drawn  
  		if the next tile is not movable (due to boundary / drawing limit), 
			nothing should be changed
			it also pushes the movement ot the op_stack
			
		- complete_lvl()
			happens when the move-to tile is an endpoint with the same color
			of the current pipe. 
			update the number of moves 
	
		
		- undraw_pipe()
			happens when the cursor is on a connected tile, this will disconnect
			the two endpoints. The connected pipe needs to be valid. 
			It also relies on the assumption that only 2 endpoints max per color.  
  		
  	
COMPONENT DISPLAY
	STATEs
		- x: 		top_left x offset
		- y: 		top_left y offset
		- w: 		width 
		- h: 		height

	ACTIONs
		- show: 		show the display on the console
		- clear: 		clear to empty 

COMPONENT INSTRUCTION: is a DISPLAY

COMPOENTN GAME_STAT: is a DISPLAY 
	STATEs
		- cur_level:		current level 
		- time_elapsed: time elaspsed this GAME 
		- running: 			if game is running / paused 
		- moves: 				number of moves from previous levels
		- level_moves: 	number of moves current level
		
	ACTIONs
		- done_level:		
			called to update the game stats when a game is finished 
		
    - timer_tick:
    	called every timer interrupt, should update the time_elasped 
    
    - restart_level:
    	clear the current level stats
    	
    - pause/resume:
    	pause/resume the timer 
    
		
		

```

```
2. Error handling?
3. Input supported 
4. Define rules 
	- how moves is calculated
	- how score is calculated 
	- how time is tracked 
	- how undo is performed 
	
Driver:
- how to handle the console (fast procesing with the ring buffer)
```







TODO: 





Console Related Design:

- keep track of color
- 



Class Notes

### Ops 

1. Atomic Instruction sequence

2. Voluntary de-scheduling 







## Project 2

vanish()

task terminations: p6

Threading Design 

- Synchronous threading vs Asynchronous threading?
- Fork() and exev() copies threads?
- signal handling 



Questions need to figure out 

1. How to do software exception handling 
 - p11 

2. When do you need software exception? 

3. When to create a new thread 


4. How to handle thread crash 
5. How to handle legacy auto stack growth 
	- use the `stack_test1 `
	- need to install a handler with `swexn() `
	- different in the multithreads program (handle the transition in `thr_init`)
6. How to write syncrhonize primitives 
7. what if a thread crashed from hardware exceptions (page fault?) ? and while holding on resources? or crashed volunatrily 



Thread API: 

1. `the_init`:

   - Set up the software exception (transition from the legacy mode to multi threading mode) 
   - :a: What data structures (with what scope) need to be setup? 

   



2. `thr_create` 

   - need to setup a stack 
     - :a: How to set up the stack? 
     - :a: What is the size of the stack 
     - :a: What if runs out of stack memory 

   - How to call the thread_fork? 
     - through asm code?
   - The new thread needs to:
     - set up software exception 
     - Set up register values because all register values other than %eax will be the same 
       - :a: what values should be set for these registers? 
     - Set up the next instruction 
   -  :a: How to make a thread exiting on `thr_exit` 
     -  Set up the return value to `thr_exit`? 

3. `thr_join`:

   - how to wait for another thread 

     - IF still running 

       - Note it down somewhere that "I am waiting for thread X"
       - Deschedule itself
       - When thread X exitted, make me runnable 

     - ELSE IF: x not exited 

     - ELSE IF: someone joining X already 

     - ELSE IF: x have already exited 

     - ELSEIF: x never initialized 

       

4. `thr_exit`:
   - This has to notifier the one calling thread join 
   - call set_status if the last thread 
   - Functins not calling it should also do a `thr_exit`
   - :a: what resources is being cleaned up and what's not?



5. `thr_getid`:
   - just call gettid() right?
   - is the thread id of the user be same as the kernel thread id? 
     - It should be a 1:1 mapping model right? If thread_create will always call into the system call. 

6. `thr_yield`:
   - deschedule itself and make_runnable others if specified 

7. How to make thread routines thread-safe? 
8. How to make thread library effecient (system call might "take a while" to run)?
   - Try not calling into the kernel? 

9. Set up a software exception handler for thread crash 
10. Where to set the initial address (stack_high / esp) 

## Reading

atomic updates on a shared data structure do not mean no race condition 



# OH 

<img src="/Users/chenxu/Library/Application Support/typora-user-images/image-20200925133937041.png" alt="image-20200925133937041" style="zoom:30%;" />

What's wrong with this code?

> going_out_of_business becomes true when cond_wait wakes up? 

# Database 

### P2

Questions: 

- DeleteEntry call signature changes with addition of RID 





# RAFT revisit

- Data only flows from leader to followers 
- Safety invariant: if one server applied a log entry, others should do the same for that log index. 



How to ensure one leader one term?



How to exit the candidate state? (3)



WHat does the follow do on receiving request votes?



When is an entry safely replciated?



What gives the log matching property (a log at an index and all logs preceiding it, if matched with term, should be identical)

- A log entry never changes its index 
- Logs consistency check (receiver checks if the log entry before new entries are found in its local logs)
  - nextIndex in the leader (keeps decrementing) until make sure followers catch up
  - An optimization (skip all wrong entries with the same incorrect term) could save number of AppendEntries calls if the follower replies the index of the first log with that non-matching log term. 



How to prevent a stale leader?

- Alternative would be allow leaders to catch up
- Raft enforces update-to-date leader during election (voters will reject if itself more updated)



Why cannot determinate commitment from previous terms (when leader recovers)?

- When recovering, even a log previous term is being replicated on majority, another server (with a more recent log) could become leader, and overwrite it. 



Consensus log entries:

- Commit entries also ensure history 
- 



Leader: 

- Should try forware entries if follower fails/network slow, EVEN after replying to client 
- include commitIndex (highest log entry committed) in messages to others





## OS 

BSD context switching 

- voluntary and involuntary context switches happen in different routines. 



### Question 1: Educational value of material

While doing our P3, I was curious how FreeBSD implements synchronization in the kernel. The section on "Context Switch" covers some really cool design and data structures. The most interesting being the "turnstile". 

I knew that a kernel thread might block for different reasons, but then came to realize that a thread will also block for a different period of time depending on the reason: a long wait might happen if a thread is waiting for an event that could only happen at an indeterminate time in the future, such as user input; a medium wait would happen if the event it is waiting on might not happen immediately but not too far in the future, such as reading from disk; and a short wait happens when a lock request is not granted (in fact, this is the only case a short wait happens in FreeBSD). 

The turnstile is what FreeBSD uses to manage a short-term lock. Each turnstile keeps track of:

1. the owner of the lock (pointer to thread's TCB)
2. pointer to the lock 
3. a list of waiters
4. pointer to other turnstiles. 

One less intuitive design regarding the turnstile is its ownership. A turnstile is owned by a thread, rather than a lock because there are way more locks than a thread in the kernel. A turnstile will be allocated at thread's creation, and passed along between different threads during execution: when a thread gets blocked on a lock, it will add itself to the waiters of the turnstile if there is already a turnstile, and add its turnstile to a free list. If it is the first thread getting blocked, it will give out its turnstile. When a thread is unblocked, it will take a turnstile from the free list if it is not the last one. 

Another functionality that the turnstile made possible is priority propagation when a thread with higher priority trying to acquire a lock owned by a thread with lower priority (priority inversion). With the owner TCB, the higher priority thread could "lend" its priority to the low-priority thread running. When the low-priority lock owner thread unlocks, it could then return to its original priority. 

A related concept to turnstile is called "wait-channel", which is a pointer that represents a resource that a thread could wait on. Lock is a kind of wait channel, and the wait channel of a lock will be used as a key into a global turnstile hash table to derive the turnstile. 

### Question 2: Recommendation

I would probably recommend this book to someone interested in knowing how a mature operating system is implemented, but not who wishes to learn operating system concepts. At first, I tried to use this book as a guide to teaching me how to write a kernel, wishing to find snippets or chapters that discuss some of the implementations in great detail, such as deadlock avoidance/prevention, process termination synchronization, kernel synchronization, etc. However, I soon realized that although the book covered such things, but not detailed enough as a "reference kernel teaching guide". After all, that might not be the most suitable usage of this book. 

But the chapters that cover security, filesystems, etc are great for anyone who wants to learn the relevant topics. Those topics, however, are not so easy to digest since the book does go into details. Overview for some of the topics like the Fast Filesystem, the Zettabyte Filesystem is not easy to understand, even though it should be a high-level discussion of the topic. 



### Question 3: Value of this assignment

I think it is great that the course staff kind of forces us to do more reading! Fortunately, my partner and I pick the same book to read, and it was from our discussion that I felt I learned the most because I tried to internalize the content and discuss those topics I read with him. So I guess maybe more frequent discussion in the format of post/"chapter report" among partners or different groups would be a good exercise to add to the book report assignment. This will also "force" us to not do binge-reading in some ways. 



What is a wait-channel, and how process wake and sleep on it?



Why are there different terms locks?

- short term for acquiring lock 
- Mid-term for sleeps / I/O
- long-term for waiting for user?



What is turnstile? 

- A data-structure used to manage short-term blocks
- It has a header that maps from the address to the locks? (Not sure what is the key of that hash header tho)
- <img src="/Users/chenxu/Library/Application Support/typora-user-images/image-20201106124338610.png" alt="image-20201106124338610" style="zoom:50%;" />
- A turnsitle is owned by the thread (since a thread blocks on at most 1 lock)
- priory inversion will be fixed with priority propogation through the turnstile waiting list 





## Memory Ordering 

- we as programmers expect program order
- processor aggresively reorders instructions to hide memory latency 
- orer of order R/W on different cache blocsk work on uni-processor, but not on multi-procesor 
  - easy for Uni-processor to have atomic memory accesses, in program order
- how does out-of-order work
  - processor fetches instructin in-order
  - issue out-of-order (preserving )

- issueing in-order in multi-processor could result in order-of-order reads (due to caching and interconnect delay)



- One can use syncrhonization points to allow regions where re-ordeing is allowed 
  - a properly synchronized program should yield the same result on a Sequentially consistent machine (eeven though the underlying hardware is not SC)
- Release consistency and Weak ordering ???



## Syncrhonization 

How does cache coherence handle load/stores sequence? 

- Test and Test-and-Set

- need to lock the cache line 
- This could be optimized with a `Test adnd Test-and-Set` optimization
  - Basically wrap the atomic xchg with a while loop 
- But unlocking would genreate a lots of traffic on the cache lock still 
  - So use something like a backoff 



- Ticket lock 
  - good 
  - more space
  - still traffic on reading the next_serving 

- Queue-based or List-based locks
  - generally good for high contention



LL/SC comparing to CMPXCHG/XCHG

- LL/SC could detect A->B->A problem 



# J1 豁免

https://www.1point3acres.com/bbs/forum.php?mod=viewthread&tid=573315&highlight=j1

https://instant.1point3acres.com/thread/662977

https://www.1point3acres.com/bbs/thread-633686-1-1.html

https://zhuanlan.zhihu.com/p/23264190



Statement of Reason 

To whom it may concern,
This letter is to request a waiver for the 2-year home residence requirement based on the 'No Objection Letter' that will be obtained from my home country (P.R. China). The reasons I would like to apply for a waiver are the followings:
I came to the U.S. as a J-1 exchange scholar in 2017 during my overseas exchange program at my college, the National University of Singapore. As a J-1 holder, I was fortunate to work for a Silicon Valley-based start-up Naya Health, and learned a great deal about entrepreneurship and technology. 

After that, in 2019, as an F-1 student, I started my Master of Science in Computer Science at Carnegie Mellon University and will be graduating in December 2020. I have obtained my Post-completion OPT and will start my practical training as a Production Engineer at Facebook Inc. Facebook, as my employer, will sponsor me during the optional practical training period, helping me acquire industrial experience.

I will apply for the No Objection Letter and my embassy will directly provide you with the `No Objection Letter` issued for me.
I appreciate your support and assistance in waiving my two-year home country residence requirement.
Sincerely,

XU Chen

12/01/2020



06/18/2016 - 08/15/2016: F-1 status (N0017000058), Stanford Unviersity summer program

08/15/2019 - Now: F-1 status (N0030638519), Carnegie Mellon University Master of Science in Computer Science



尊敬的驻美总领馆：

按照《J-1签证豁免申请办法（修订稿）》的要求，特此提出豁免本人的J-1两年回国服务要求的申请。本人于2019年5月获新加坡国立大学计算机学士文凭，2019年8月自费赴美学习至今。本人在2017年1月至2017年12月参加了新加坡国立大学海外交流项目，以J1访问学者身份在美国硅谷创业公司Naya Health实习。
本人即将于2020年12月从美国卡内基梅隆大学毕业，已申请Post OPT并获批，将入职位于加利福尼亚州的Facebook工作深造。本人在海外留学期间一直是官方留学生学生联合会成员（新加坡学联，卡内基学联），受到了领事馆的不少照顾和指点。尤其是在疫情期间，收到来自外交部的抗疫资源，很是感动。本人希望能够在回国长期发展之前，在美国的科技公司深造一段时间，为之后回国的发展和贡献打下一个更加坚实的基础，特此提出J-1豁免申请。

附：留学时间及经历
2015年8月  - 2016年6月：就读新加坡国立大学
2016年6月 - 2016年8月：F-1签证，自费赴美斯坦福大学暑期学校项目
2016年8月  - 2017年1月：就读新加坡国立大学
2017年1月- 2017年12月：J-1签证，以访问学者身份自费赴美Naya Health带薪实习，并在斯坦福大学就读。
2017年12月- 2018年1月： 空闲在家（浙江杭州）
2018年1月- 2018年5月：就读新加坡国立大学
2018年5月- 2018年8月：北京微软亚洲研究院实习
2018年8月- 2019年5月：就读新加坡国立大学
2019年8月- 2019年11月：F-1签证，自费赴美卡内基梅隆大学就读计算机硕士。
2019年11月- 2020年1月：空闲在家（浙江杭州）
2020年1月- 至今：就读卡内基梅隆大学

当前所持EAD 的有效时间为：2022年1月10日

 



# OS P4 

Trap and emulate approach

- Guest kernel GP fault, Host kernel inspects during GP handler
- Guest kernel PF (writing to console memory), Host kernel handles it. 
- Requires the fault handlr **disassemble** fault address to understand what the guest kernel wants to do! :dizzy_face:



Paravirtualization approach

- rewrite part of the guest kernel so that instructions that would be trap-and-emulated now will go down a special function call 
- Interface allows bundle multiple special instructions into a single call (e.g. set_timer_rate())



Some requirements:

- no need for dual-kernel even if it is possible 



Booting guest kernels

- need to setup **Boot VM**



Interrupting the guest :

|              | physical interrupt (timer)                                   | exception (divide by zero) |
| ------------ | ------------------------------------------------------------ | -------------------------- |
| Guest User   | guest user handled to guest kernel. Host kernel needs to be in the picture? | guest user killed          |
| Guest Kernel |                                                              |                            |
|              |                                                              |                            |

- Not to interrupt the guest kernel if the guest kernel disabled interrupt 
- all guest kernel IDT entries are now interrupt gate 



hyper calls:

- `hv_iret`: what kind of checking fot eh ELFAGs 





Questions: 

- How to set the segmentation for guest kernel 
  - construct descriptors in the GDT for the guest kernel
  - max size of virtual address 





Things to happen: 

- Host: check text region in `exec_handler` 
- Direct map the guest virtual frame to guest physical frame (for guest kernel only, not guest user right?)

- Host: allocate guest physical pages (directly mapped the entire guest kernel?)
- Set those register values 
- Guest kernel: `hv_setpd` 
- HOW TO GO TO GUEST KERNEL? some `gk_iret`



Questions:

- How to enter guest kernel mode?
- 

Things to track:

- guest kernel size?



Gues kernel vm related: 

- GVA -> GPA is fixed, so given a GPA -> GVA, then get VA, 
- probably don't need to maintain the guest physical to physical mapping 





```
hv_magic: 
- return the magic number 


hv_disable_interrupts: 
- TODO 


hv_enable_interrpts: 
- TODO 


hv_setidt: 
- modify the virtual IDT 


hv_setpd:
- get the linear virtual address pdbase(pdbase:V)
- map the process pd according to the pdbase
	- traverse the pdbase:V, for each mapped pde, pde:V, 
		- get the guest physical address of the page table, pt:GF
    - pt:GF should be directly mapped in guest's address space, so we also have pt:GV,
    - translate that to linear virtual address pt:V, 
    - for each present pte:V entry in that page table (read that page table)
    	- reconstruct the linear virtual address of pte (V)
    	- check if there is the (GF) mapped to derive a HF
    	- [allocate a GF -> HF mapping if not?]
    	- update V -> HF in the uvm->pd 

hv_adjustpg(addr):
- addr is GV
- get the linear virtual V
- walk the guest's pdbase:V to get the GF address, and pte flags
- check the mapping GF -> HF
- use HF to update the uvm->pb with the same pte flags 

hv_iret:
- TODO
```





# Dario's Post 

How Dario approach works?

1. Study qualitatively and quantitativley like a Doctor / Historian 
2. Form a high level archetypical POV as hypothesis 
3. Put them into algorithms for testing 



Dario thinks there are two things in the current status-quo:

1. the static: what exists 
2. the dynamic: timeless and universal forces that produce changes 

One should be focusing on 2 rather 1. (Or at least he is)



Having the state and the central goverment fight over relative powers is a marker for entering the Stage 6 (civil war)





# IPC and RPC 

Communication and synchronization pattern: 

- blocking send
  - Good for request/response pattern 
  - Bad for producer and consumer pattern (long waiting time)
- non-blocking send
- blocking recv
  - good for 'server thread' / req+res pattern 
  - bad for handling idle client
- Non-blocking recv
  - good for polling pattern (which could be costly though)
- Timeout recv: 
  - good performance
  - but not stable across varying timer rates 



The buffering problem!!!!!:a: 



Marshalling

- recursive data marshaling structs 
- BUT implementation better to be iterative (don't have functions call short functions call short functions....)
- Marhsalling usually takes multiple pass to use the memory 



Segments 

G: needs to be 1: 4KB granularity 

DPL: 3 

P : 1

D/B flag:

	- Code: 1: so that 32-bit addresses assumued 
	- Stack: 1 for 32-bit stack pointer 



Type field:

- Code: 1 0 1 0
- Data  0 0 1 0



# Snowflake talk 

### Snowflow data storage architecture:

- A table is partitioned into multiple files
- Each file contains multiple rows 
- A meta data object is generated for each file, including statistics (range) for each column of tuples stored in the file 
- File is not modifiable! 



An insert into the table:

- would create a new file (and meta object) for those rows inserted 



A delete/update of a row: 

- would create a new file reflecting the updated and deleted row in the file 
-  would create a new meta object reflecting that the old file containing the row is not deleted. 
- this is a rare workload pattern for OLAP actually, so this is not optimized



A select:

- would scan the meta objects
- Construct and find a set of files need to scan based on the select predicate
- This is the dominant workload, so this is optimized heavily 



### Reclustering 

The select performs best if a file contains a small range of values (like the files do not overlap with each other in range, identical and adjacent values are stored in a file) 

- Therfore a select could skip those files not meeting the predicate. 



background reclustering task based on depth(overlapping number of files at specific range )

reclusting with levels 



| Record Number | Name               | Value              | Type | TTL      |
| ------------- | ------------------ | ------------------ | ---- | -------- |
| R1            | c.root-servers.net | 192.33.4.12        | A    | 24 hours |
| R2            | .                  | c.root-servers.net | NS   | 24 hours |



| Record Number | Name       | Value      | Type | TTL      |
| ------------- | ---------- | ---------- | ---- | -------- |
| R3            | b.gtld.net | 198.31.2.8 | A    | 12 hours |
| R4            | com.       | b.gtld.net | NS   | 12 hours |





| Record Number | Name          | Value          | Type | TTL     |
| ------------- | ------------- | -------------- | ---- | ------- |
| R5            | takemypaw.com | 192.102.111.5  | A    | 2 hours |
| R6            | findpet.com   | 192.102.111.6  | A    | 4 hours |
| R7            | petpet.com    | 192.102.111.11 | A    | 2 hours |



| Record Number | Name               | Value              | Type | TTl      |
| ------------- | ------------------ | ------------------ | ---- | -------- |
| R1            | c.root-servers.net | 192.33.4.12        | A    | 24 hours |
| R2            | .                  | c.root-servers.net | NS   | 24 hours |
| R8            | b.gtld.net         | 198.31.2.8         | A    | 12 hours |
| R9            | com.               | b.gtld.net         | NS   | 12 hours |
| R10           | takemypaw.com      | 192.102.111.5      | A    | 2 hours  |



# HW OS 

Instructions: 

```
mkdir -m 0700 -p /tmp/$USER/gpg-agent
eval `/usr/bin/gpg-agent --pinentry-program /usr/bin/pinentry-curses --homedir /tmp/$USER/gpg-agent --daemon`

```





# TiKV 

Things to explore further: 

- Coprocessor
- Placement Driver 



### LSM Tree 

- https://github.com/google/leveldb/blob/master/doc/impl.md



### TiKV and RockDB

https://tikv.github.io/deep-dive-tikv/key-value-engine/rocksdb.html

- Use of prefix bloom filter: 
  - false positive on keys with the same prefix 
    - MVCC in TiKV and each version consits of key (for a row) + timestamp (version info). So a PBF tells u which data region MIGHT contain the key 



### Isolation level of TiKV

It uses SI (therefore it is subject to write skew anomaly): https://tikv.github.io/deep-dive-tikv/distributed-transaction/isolation-level.html#snapshot-isolation

>As a concrete example, imagine V1 and V2 are two balances held by a single person, James. The bank will allow either V1 or V2 to run a deficit, provided the total held in both is never negative (i.e. V1 + V2 ≥ 0). Both balances are currently $100. James initiates two transactions concurrently, T1 withdrawing $200 from V1, and T2 withdrawing $200 from V2.
>
>If the database guaranteed serializable transactions, the simplest way of coding T1 is to deduct $200 from V1, and then verify that V1 + V2 ≥ 0 still holds, aborting if not. T2 similarly deducts $200 from V2 and then verifies V1 + V2 ≥ 0. Since the transactions must serialize, either T1 happens first, leaving V1 = -$100, V2 = $100, and preventing T2 from succeeding (since V1 + (V2 - $200) would be -$200), or T2 happens first and similarly prevents T1 from committing.
>
>If the database is under snapshot isolation (MVCC), however, T1 and T2 operate on private snapshots of the database: each deducts $200 from an account, and then verifies that the new total is zero, using the other account value that held when the snapshot was taken. Since neither update conflicts, both commit successfully, leaving V1 = V2 = -$100, and V1 + V2 = -$200.



### Tmestamp Oracle 

- a central service that gave out timestamps in increasing order 
- the highest range of tiemstamps is persisted to the disk so recovery will continue the timestamps from that 
- requests batched to save RPC overhead (at the cost of higher txn latency) 
- Lock -> request commit timestamp -> Write, and request start timestamp -> Read, these happens before relationship make sure that a lower timestamp will guarantee READ COMMITTED 



### Service Layer 

Jobs: 

- RPC service 
- Store id => address mapping 
- TIKV communication 



### Storage Layer 

https://pingcap.com/blog-cn/tikv-source-code-reading-11/

- stores transaction and MVCC stuff 
- execution engines uses function such as `async_read` and `async_write` as API into the storage engines 
- Has raw and transactional APIs 
- There are BTree engine, RocksDBEngine (this seems to be the one used)



### Distributed Transaction 

- Cross regional transaction provided at TiDB, TiKV only suports single region transaction (client, `TiDB's store/tikv` needs to handle multiple regions )

##### The `Lock` mutation type: 

- it is used for ensure some keys not to be changed during tranx (like `SELECT .. FOR UPDATE` )

##### The `Rollback` mutation type:

- Rollback's commit_ts is the same as start_ts, so that a prewrite from the same transaction would be visible, and the prewrite will fail on write conflict.
- Rollback will happen if a txn's request timeout, and rollbacked by other transactions. 

##### Remaining  Locks resolution 

- If a lock has timeout, check the owner's status. If the owner committed, obtain its `commit_ts` to clear its other secondary locks. If the owner not committed yet, roll it back 
- lock resolution comes with the lite version and the full version:
  - lite: only clears those keys specified 
  - full: all locks in the same region with `start_ts` equal to a certain `ts`
- TiKV uses latches to lock all the keys 



##### Sequential Forward Scan

https://pingcap.com/blog-cn/tikv-source-code-reading-13/

- Keeps the Write and Lock cursor. 
- Not sure why the lock cursor check is this way....



### Coprocessor 

https://pingcap.com/blog-cn/tikv-source-code-reading-14/

##### Motivation

Push physical plan to TiKV (Storage) so that computation could be done closer to data 





# Percolator

#### Motivation 

- continously transforming a large repo of documents as new documents arrive (indexing all the pages) 
- Alternative: batch-based indexing using MapReduce framework
  - Slow: proportional to the size of all the docs 
- Alternative: traditional DBMS
  - Not scalable: petabytes of data in 



#### High level features :

- Observers: user-speicifed column changes callbacks 
- ACID transactions(multiple-row) over a random-access repo 



#### Design

- Each node consists of : 
  - percolator worker 
  - Bigtable tablet server
  - GFS
- Uses timestamp oracle and lighweight lock service 
- No central transaction management service 



### Locking: 

- stores lock in special in-meemory columns 

##### Writing multiple rows: 

- update 1st: (ACID transaction to single row providide by Bigtable)
  - add entry to the data column 
  - add primary lock to the lock column 
- update another row:
  - add entry to the data column 
  - add secondary lock to the lock column 
- Commit:
  - add to the write column update 1 for commit
  - reomve the primary lock 
  - (Optional?) add to the write column of update 2 and remove the secondary lock 



#### Evaluation:

- **Reduces High latency:**  incremental so no more batching updates necessary. (50% lower average age of docs in Google search results => more recent indexed results )
- **NOT good for large computation**: those are better for MapReduce framework, good for incremental small changes









# TA meeting 

raw images in the gdrive 

points allocation for eahc question and total assignment 







# Intern Prcticum 

During this year's summer, I worked at Facebook as a Production Engineering Intern under the guidance of Ryan Barber (r00t@fb.com). I worked with the Presto team, which maintains the Presto, a distributed query exeuction engine. The main goal of my project was to rewrite the query replay testing pipeline. The query replay tool is used as part of the testing pipeline to ensure a set of queries could be replayed at any cluster with the expected outcome and performance. 

The existing query replay framework was built on top of legacy technology that is resource consuming. While supporting multiple queries to be replayed concurrently, the old tool consumes substantial amount of resources by launching a process for each query. My project invovles rewritting the tool so that it supports all the legacy usage (flags), but also facilitates the benchmarking usage by collecting metrics and statistics. 

The project is primarily written in Python, and I used the asyncio library heavily for the new implementation. Other than the technical obstacles which required me to architecture the new tool with multiple event loops to be both scalable and effecient, the greatest lessons was on how to best do communication. There was definitely the need to communicate with collegues on various aspects of the project, such as usage requirement, feature request, bug report, and etc. More importantly, there was also the sharing of progress and result. Similar to what I did with school projects, I only thought of making a post about my final result when the internship ends, as a wrap-up. But apparently, I could have made more incremental updates on the project, or saught earlier feedbacks when possible. 

Looking back at the internship, I wish I could have stepped out of the scope of my project and took other responsibility earlier. Also, I could have been more proactive, in my communication with the rest of the team, and in learning other relevant products and projects. Facebook has great infrastructures to ensure its product and service run reliably and effeciently, and I wish I had spent more times digging into the internals of other tools. 









# DB

- Logical composition 



How is a query template like?

When would cluster change in one day or two?





# Questions to ask 

Why we don't have EINTR as part of the design in our Pebbles kernel?



