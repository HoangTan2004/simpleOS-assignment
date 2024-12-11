# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment

Time slot   0
ld_routine
	Loaded a process at input/proc/a1, PID: 1 PRIO: 0
Time slot   1
	CPU 0: Dispatched process  1
Time slot   2
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 1: rg_start 0, rg_end 300, vmaid 0
vma0: vm_start 0, vm_end 511
Count free list: 0
vma1: vm_start 2559, vm_end 2559
Count free list: 0
-----------------------------------------
Time slot   3
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Time slot   4
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 4: rg_start 300, rg_end 600, vmaid 0
vma0: vm_start 0, vm_end 767
Count free list: 0
vma1: vm_start 2559, vm_end 2559
Count free list: 0
-----------------------------------------
Time slot   5
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Time slot   6
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 2: rg_start 600, rg_end 1000, vmaid 0
vma0: vm_start 0, vm_end 1023
Count free list: 0
vma1: vm_start 2559, vm_end 2559
Count free list: 0
-----------------------------------------
Time slot   7
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Time slot   8
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 4: rg_start 300, rg_end 600, vmaid 0
vma0: vm_start 0, vm_end 1023
Count free list: 1
vma1: vm_start 2559, vm_end 2559
Count free list: 0
-----------------------------------------
Time slot   9
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Time slot  10
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 1: rg_start 0, rg_end 300, vmaid 0
vma0: vm_start 0, vm_end 1023
Count free list: 2
vma1: vm_start 2559, vm_end 2559
Count free list: 0
-----------------------------------------
Time slot  11
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Time slot  12
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 3: rg_start 1000, rg_end 1400, vmaid 0
vma0: vm_start 0, vm_end 1535
Count free list: 2
vma1: vm_start 2559, vm_end 2559
Count free list: 0
-----------------------------------------
Time slot  13
	CPU 0: Processed  1 has finished
	CPU 0 stopped


0 12
alloc 300 1
dump 1
alloc 300 4
dump 4
alloc 400 2
dump 2
free 4
dump 4
free 1
dump 1
alloc 400 3
dump 3
alloc 100 0	
dump 0
