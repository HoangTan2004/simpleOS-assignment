# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment




int inc_vma_limit(struct pcb_t *caller, int vmaid, int inc_sz, int* inc_limit_ret) {
    struct vm_rg_struct *newrg = malloc(sizeof(struct vm_rg_struct));
    newrg->vmaid = vmaid;

    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    int old_sbrk = cur_vma->sbrk;
    int inc_amt;

    // Xử lý vmaid == 0
    if (vmaid == 0) {
        if (cur_vma->vm_end == 0) {
            old_sbrk = -1; // Nếu vm_end là 0, gán old_sbrk là -1
        }
        // Tính toán vm_end mới cho vmaid == 0
        int new_vmaend = PAGING_PAGE_ALIGNSZ(old_sbrk + inc_sz) - 1;
        inc_amt = PAGING_PAGE_ALIGNSZ(-cur_vma->vm_end + new_vmaend);
    }
    // Xử lý vmaid != 0 (có thể là heap, stack, v.v)
    else {
#ifdef MM_PAGING_HEAP_GODOWN
        if (cur_vma->vm_end == caller->vmemsz - 1)
            old_sbrk = caller->vmemsz;  // Nếu vm_end là cuối cùng của bộ nhớ, gán old_sbrk bằng vmemsz
        int new_vmaend = ((old_sbrk - inc_sz) / PAGING_PAGESZ) * PAGING_PAGESZ;
        inc_amt = PAGING_PAGE_ALIGNSZ(cur_vma->vm_end - new_vmaend);
#endif
    }

    int incnumpage = inc_amt / PAGING_PAGESZ; // Tính số trang cần cấp phát

    struct vm_rg_struct *area = get_vm_area_node_at_brk(caller, vmaid, inc_sz, inc_amt);
    int old_end = (int)cur_vma->vm_end;

    // Kiểm tra xem vùng bộ nhớ có bị chồng lấn không
    if (validate_overlap_vm_area(caller, vmaid, area->rg_start, area->rg_end) < 0) {
        free(newrg);
        free(area);
        return -1;  // Nếu có chồng lấn, trả về lỗi
    }

    // Cập nhật vm_end và sbrk
    if (vmaid == 0) {
        cur_vma->vm_end = PAGING_PAGE_ALIGNSZ(old_sbrk + inc_sz) - 1;
    } else {
        cur_vma->vm_end = ((old_sbrk - inc_sz) / PAGING_PAGESZ) * PAGING_PAGESZ; // Điều chỉnh cho vùng heap
    }

    // Ánh xạ bộ nhớ mới vào RAM
    if (vm_map_ram(caller, area->rg_start, area->rg_end, old_end, incnumpage, newrg) < 0) {
        cur_vma->vm_end = old_end;  // Nếu ánh xạ thất bại, hoàn nguyên
        free(area);
        free(newrg);
        return -1;
    }

    free(area);
    free(newrg);

    *inc_limit_ret = inc_amt;  // Trả về kích thước bộ nhớ đã thay đổi
    return 0;
}
