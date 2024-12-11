# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment




/*__alloc - allocate a region memory
 *@caller: caller
 *@vmaid: ID vm area to alloc memory region
 *@rgid: memory region ID (used to identify variable in symbole table)
 *@size: allocated size 
 *@alloc_addr: address of allocated memory region
 *
 */
int __alloc(struct pcb_t *caller, int vmaid, int rgid, int size, int *alloc_addr) {
    // Kiểm tra nếu vùng đã được cấp phát
    if (caller->mm->symrgtbl[rgid].rg_start != caller->mm->symrgtbl[rgid].rg_end) {
      return -1;
    }

    struct vm_rg_struct rgnode;
    rgnode.vmaid = vmaid;

    // Thử tìm vùng trống trong danh sách vùng nhớ
    if (get_free_vmrg_area(caller, vmaid, size, &rgnode) == 0) {
      // Gán vùng nhớ vào bảng ký hiệu
      caller->mm->symrgtbl[rgid].rg_start = rgnode.rg_start;
      caller->mm->symrgtbl[rgid].rg_end = rgnode.rg_end;
      caller->mm->symrgtbl[rgid].vmaid = rgnode.vmaid;

      // Khởi tạo giá trị mặc định trong vùng nhớ
      for (int addr = rgnode.rg_start; addr < rgnode.rg_end; addr++) {
          pg_setval(caller->mm, addr, '\0', caller);
      }

      *alloc_addr = rgnode.rg_start;
      return 0;
    }

    // Nếu không tìm thấy vùng trống, mở rộng giới hạn vma
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
      return -1;  
    }
    
    int old_sbrk = cur_vma->sbrk;
    int new_sbrk = (vmaid == 0) ? old_sbrk + size : old_sbrk - size;
    int inc_limit_ret = 0;
    
    if ((vmaid == 0 && new_sbrk > cur_vma->vm_end) || (vmaid != 0 && new_sbrk < cur_vma->vm_end)) {
      if (inc_vma_limit(caller, vmaid, size, &inc_limit_ret) < 0) {
          return -1;
      }
    }
    
    caller->mm->symrgtbl[rgid].rg_start = old_sbrk;
    caller->mm->symrgtbl[rgid].rg_end = new_sbrk;
    caller->mm->symrgtbl[rgid].vmaid = rgnode.vmaid;
    cur_vma->sbrk = new_sbrk;

    // Khởi tạo giá trị mặc định trong vùng nhớ
    for (int addr = old_sbrk; addr != new_sbrk; addr += (vmaid == 0 ? 1 : -1)) {
      pg_setval(caller->mm, addr, '\0', caller);
    }

    *alloc_addr = old_sbrk;
    return 0;
}

/*__free - remove a region memory
 *@caller: caller
 *@vmaid: ID vm area to alloc memory region
 *@rgid: memory region ID (used to identify variable in symbole table)
 *@size: allocated size 
 *
 */
int __free(struct pcb_t *caller, int rgid)
{
  struct vm_rg_struct *rgnode = malloc(sizeof(struct vm_rg_struct));

  if (rgid < 0 || rgid > PAGING_MAX_SYMTBL_SZ)
    return -1;

  /* TODO: Manage the collect freed region to freerg_list */
  rgnode->rg_start = caller->mm->symrgtbl[rgid].rg_start;
  rgnode->rg_end = caller->mm->symrgtbl[rgid].rg_end;
  rgnode->vmaid = caller->mm->symrgtbl[rgid].vmaid;
  
  //enlist_vm_freerg_list(caller->mm, rgnode);
  memset(&caller->mm->symrgtbl[rgid], 0, sizeof(struct vm_rg_struct));
  return 0;
}
