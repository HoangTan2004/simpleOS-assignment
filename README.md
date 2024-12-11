# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment



int inc_vma_limit(struct pcb_t *caller, int vmaid, int inc_sz, int *inc_limit_ret) {
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1;  // Không tìm thấy VMA
    }

    int old_sbrk = cur_vma->sbrk;
    int old_end = cur_vma->vm_end;
    int inc_amt;

    // Xử lý trường hợp vmaid == 0 (heap tăng)
    if (vmaid == 0) {
        if (cur_vma->vm_end == 0) {
            old_sbrk = -1;  // Nếu vm_end là 0, đặt old_sbrk = -1
        }
        int new_vmaend = PAGING_PAGE_ALIGNSZ(old_sbrk + inc_sz) - 1;
        inc_amt = PAGING_PAGE_ALIGNSZ(new_vmaend - cur_vma->vm_end);
    } else {
#ifdef MM_PAGING_HEAP_GODOWN
        // Xử lý heap giảm dần
        if (cur_vma->vm_end == caller->vmemsz - 1) {
            old_sbrk = caller->vmemsz;
        }
        int new_vmaend = ((old_sbrk - inc_sz) / PAGING_PAGESZ) * PAGING_PAGESZ;
        inc_amt = PAGING_PAGE_ALIGNSZ(cur_vma->vm_end - new_vmaend);
#endif
    }

    int incnumpage = inc_amt / PAGING_PAGESZ;

    struct vm_rg_struct *area = get_vm_area_node_at_brk(caller, vmaid, inc_sz, inc_amt);
    if (!area) {
        return -1;  // Lỗi cấp phát
    }

    // Kiểm tra chồng lấn
    if (validate_overlap_vm_area(caller, vmaid, area->rg_start, area->rg_end) < 0) {
        free(area);
        return -1;
    }

    // Cập nhật vm_end
    if (vmaid == 0) {
        cur_vma->vm_end = PAGING_PAGE_ALIGNSZ(old_sbrk + inc_sz) - 1;
    } else {
        cur_vma->vm_end = ((old_sbrk - inc_sz) / PAGING_PAGESZ) * PAGING_PAGESZ;
    }

    // Ánh xạ bộ nhớ vào RAM
    if (vm_map_ram(caller, area->rg_start, area->rg_end, old_end, incnumpage, area) < 0) {
        cur_vma->vm_end = old_end;  // Nếu ánh xạ thất bại, hoàn nguyên
        free(area);
        return -1;
    }

    // Cập nhật kích thước đã tăng
    *inc_limit_ret = inc_amt;
    free(area);
    return 0;
}

