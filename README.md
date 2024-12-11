# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment


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
    
    int inc_sz = PAGING_PAGE_ALIGNSZ(size);
    int old_sbrk = cur_vma->sbrk;
    int inc_limit_ret = 0;

    if (vmaid == 0) {  // Vùng nhớ DATA/STACK
      if (old_sbrk + size > cur_vma->vm_end) {
        if (inc_vma_limit(caller, vmaid, size, &inc_limit_ret) < 0) {
          return -1;
        }
      }

      // Cập nhật bảng ký hiệu
      caller->mm->symrgtbl[rgid].rg_start = old_sbrk;
      caller->mm->symrgtbl[rgid].rg_end = old_sbrk + size;
      cur_vma->sbrk = old_sbrk + size;
    } else {  // Vùng nhớ HEAP
      if (old_sbrk - size < cur_vma->vm_end) {
        if (inc_vma_limit(caller, vmaid, size, &inc_limit_ret) < 0) {
          return -1;
        }
      }

      // Cập nhật bảng ký hiệu
      caller->mm->symrgtbl[rgid].rg_start = old_sbrk;
      caller->mm->symrgtbl[rgid].rg_end = old_sbrk - size;
      cur_vma->sbrk = old_sbrk - size;
    }

    caller->mm->symrgtbl[rgid].vmaid = vmaid;

    // Khởi tạo giá trị mặc định trong vùng nhớ
    for (int addr = caller->mm->symrgtbl[rgid].rg_start;
      addr != caller->mm->symrgtbl[rgid].rg_end;
      addr += (vmaid == 0 ? 1 : -1)) {
      pg_setval(caller->mm, addr, '\0', caller);
    }

    *alloc_addr = caller->mm->symrgtbl[rgid].rg_start;
    return 0;
}
