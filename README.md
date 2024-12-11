# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment

int inc_vma_limit(struct pcb_t *caller, int vmaid, int inc_sz, int *inc_limit_ret) {
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1; // Không tìm thấy VMA
    }

    int old_sbrk = cur_vma->sbrk;
    int new_sbrk = old_sbrk - inc_sz;

    // Kiểm tra giới hạn
    if (new_sbrk < cur_vma->vm_end) {
        return -1; // Vượt quá giới hạn heap
    }

    // Cập nhật `sbrk` và `vm_end`
    cur_vma->sbrk = new_sbrk;
    *inc_limit_ret = inc_sz;
    return old_sbrk; // Trả về điểm bắt đầu cấp phát
}


int get_free_vmrg_area(struct pcb_t *caller, int vmaid, int size, struct vm_rg_struct *newrg) {
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1; // Không tìm thấy VMA
    }

    struct vm_rg_struct *rgit = cur_vma->vm_freerg_list;

    while (rgit) {
        if (rgit->rg_start - size >= rgit->rg_end) {
            // Vùng trống đủ lớn
            newrg->rg_start = rgit->rg_start;
            newrg->rg_end = rgit->rg_start - size;

            // Cập nhật vùng còn lại trong danh sách
            rgit->rg_start -= size;
            if (rgit->rg_start == rgit->rg_end) {
                // Nếu vùng còn lại trống hoàn toàn, xóa nút khỏi danh sách
                cur_vma->vm_freerg_list = rgit->rg_next;
                free(rgit);
            }

            return 0; // Thành công
        }

        rgit = rgit->rg_next;
    }

    return -1; // Không tìm thấy vùng trống
}
