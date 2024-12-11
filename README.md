# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment

Time slot   0
ld_routine
	Loaded a process at input/proc/a1, PID: 1 PRIO: 0
	CPU 0: Dispatched process  1
Time slot   1
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Time slot   2
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
write region=1 offset=192 value=15
print_pgtbl for vmaid 0: 0 - 3583
00000000: 8000000c
00000004: 8000000b
00000008: 8000000a
00000012: 80000009
00000016: 80000008
00000020: 80000007
00000024: 80000006
00000028: 80000005
00000032: 80000004
00000036: 80000003
00000040: 80000002
00000044: 80000001
00000048: 8000000d
00000052: 8000000e
print_pgtbl for vmaid 1: 3145727 - 3145727
----------------MEMPHY DUMP--------------
-----------------------------------------
Time slot   3
	CPU 0: Put process  1 to run queue
	CPU 0: Dispatched process  1
Statistic system VMA---------------------
rgid 1: rg_start 3123, rg_end 3473, vmaid 0
vma0: vm_start 0, vm_end 3583
Count free list: 0
vma1: vm_start 3145727, vm_end 3145727
Count free list: 0
-----------------------------------------
Time slot   4
	CPU 0: Processed  1 has finished
	CPU 0 stopped


1 1 1
1048576 16777216 0 0 0 3145728
0 a1 0

1 4
alloc 3123 0
alloc 350 1
write 15 1 192
dump 1
