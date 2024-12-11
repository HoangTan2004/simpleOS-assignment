# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment

int inc_vma_limit(struct pcb_t *caller, int vmaid, int inc_sz, int *inc_limit_ret) {
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1; // Không tìm thấy VMA
    }

    struct vm_rg_struct *free_node = cur_vma->vm_freerg_list;
    int found = 0;
    int new_start = 0;

    // Kiểm tra nếu có vùng trống phù hợp trong danh sách
    while (free_node) {
        if ((free_node->rg_start - inc_sz >= free_node->rg_end) && (vmaid != 0)) {
            // Heap giảm: chọn vùng trống lớn hơn hoặc bằng yêu cầu
            found = 1;
            new_start = free_node->rg_start;
            free_node->rg_start -= inc_sz; // Cập nhật vùng còn lại
            break;
        }
        free_node = free_node->rg_next;
    }

    if (!found) {
        // Nếu không tìm thấy vùng trống phù hợp, giảm `sbrk`
        int old_sbrk = cur_vma->sbrk;

        if (vmaid == 0) {
            return -1; // Không hỗ trợ heap tăng trong trường hợp này
        }

#ifdef MM_PAGING_HEAP_GODOWN
        int new_sbrk = old_sbrk - inc_sz;
        if (new_sbrk < cur_vma->vm_end) {
            return -1; // Vượt quá giới hạn heap
        }

        cur_vma->sbrk = new_sbrk; // Cập nhật `sbrk`
        new_start = old_sbrk;    // Vùng mới bắt đầu từ `old_sbrk`
#endif
    }

    *inc_limit_ret = inc_sz;
    return new_start;
}


int __free(struct pcb_t *caller, int rgid) {
    if (rgid < 0 || rgid >= PAGING_MAX_SYMTBL_SZ) {
        return -1; // ID không hợp lệ
    }

    struct vm_rg_struct *rgnode = malloc(sizeof(struct vm_rg_struct));
    if (!rgnode) {
        return -1; // Lỗi cấp phát bộ nhớ
    }

    rgnode->rg_start = caller->mm->symrgtbl[rgid].rg_start;
    rgnode->rg_end = caller->mm->symrgtbl[rgid].rg_end;

    // Đặt lại bảng ký hiệu
    caller->mm->symrgtbl[rgid].rg_start = -1;
    caller->mm->symrgtbl[rgid].rg_end = -1;

    // Thêm vùng giải phóng vào danh sách vùng trống
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, rgnode->vmaid);
    if (!cur_vma) {
        free(rgnode);
        return -1;
    }

    enlist_vm_rg_node(&cur_vma->vm_freerg_list, rgnode);
    return 0;
}


int __alloc(struct pcb_t *caller, int vmaid, int rgid, int size, int *alloc_addr) {
    if (caller->mm->symrgtbl[rgid].rg_start != caller->mm->symrgtbl[rgid].rg_end) {
        return -1; // Vùng đã được cấp phát
    }

    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1; // Không tìm thấy VMA
    }

    int inc_limit_ret = 0;
    int new_start = inc_vma_limit(caller, vmaid, size, &inc_limit_ret);

    if (new_start < 0) {
        return -1; // Không thể cấp phát
    }

    // Gán vùng nhớ vào bảng ký hiệu
    caller->mm->symrgtbl[rgid].rg_start = new_start;
    caller->mm->symrgtbl[rgid].rg_end = new_start - size;
    caller->mm->symrgtbl[rgid].vmaid = vmaid;

    *alloc_addr = new_start;
    return 0;
}
