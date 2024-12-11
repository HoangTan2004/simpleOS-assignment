# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment




int get_free_vmrg_area(struct pcb_t *caller, int vmaid, int size, struct vm_rg_struct *newrg) {
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1;  // Không tìm thấy VMA
    }

    struct vm_rg_struct *rgit = cur_vma->vm_freerg_list;
    while (rgit) {
        // Kiểm tra nếu vùng đủ lớn
        if ((rgit->rg_end - rgit->rg_start) >= size) {
            newrg->rg_start = rgit->rg_start;
            newrg->rg_end = rgit->rg_start + size;
            newrg->vmaid = vmaid;

            // Cập nhật lại danh sách vùng trống
            if (rgit->rg_end == newrg->rg_end) {
                // Nếu dùng hết vùng trống hiện tại, bỏ nút này khỏi danh sách
                cur_vma->vm_freerg_list = rgit->rg_next;
                free(rgit);
            } else {
                // Cập nhật vùng trống còn lại
                rgit->rg_start += size;
            }

            return 0;
        }
        rgit = rgit->rg_next;
    }

    return -1;  // Không tìm thấy vùng trống phù hợp
}
