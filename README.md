# <img src="https://upload.wikimedia.org/wikipedia/commons/f/f0/HCMCUT.svg" alt="HCMUT" width="23" /> simpleOS-assignment




int inc_vma_limit(struct pcb_t *caller, int vmaid, int inc_sz, int *inc_limit_ret) {
    // Lấy thông tin VMA hiện tại
    struct vm_area_struct *cur_vma = get_vma_by_num(caller->mm, vmaid);
    if (!cur_vma) {
        return -1;  // Không tìm thấy VMA
    }

    int old_sbrk = cur_vma->sbrk;
    int old_end = cur_vma->vm_end;
    int new_end, inc_amt;

    // Tính toán kích thước tăng cho heap tăng hoặc giảm
    if (vmaid == 0) {
        // Heap tăng lên
        new_end = PAGING_PAGE_ALIGNSZ(old_sbrk + inc_sz) - 1;
        inc_amt = PAGING_PAGE_ALIGNSZ(new_end - old_end);
    } else {
        // Heap giảm xuống
#ifdef MM_PAGING_HEAP_GODOWN
        if (old_sbrk == caller->vmemsz) {
            old_sbrk = caller->vmemsz;  // Sửa lại nếu sbrk đang ở giới hạn cuối bộ nhớ
        }
        new_end = ((old_sbrk - inc_sz) / PAGING_PAGESZ) * PAGING_PAGESZ;
        inc_amt = PAGING_PAGE_ALIGNSZ(old_end - new_end);
#endif
    }

    // Tính số trang cần cấp phát
    int incnumpage = inc_amt / PAGING_PAGESZ;

    // Lấy vùng bộ nhớ từ brk
    struct vm_rg_struct *area = get_vm_area_node_at_brk(caller, vmaid, inc_sz, inc_amt);
    if (!area) {
        return -1;  // Lỗi cấp phát vùng bộ nhớ
    }

    // Kiểm tra chồng lấn vùng nhớ
    if (validate_overlap_vm_area(caller, vmaid, area->rg_start, area->rg_end) < 0) {
        free(area);
        return -1;  // Nếu có chồng lấn, trả về lỗi
    }

    // Cập nhật vm_end cho heap
    cur_vma->vm_end = new_end;

    // Ánh xạ bộ nhớ vào RAM
    if (vm_map_ram(caller, area->rg_start, area->rg_end, old_end, incnumpage, area) < 0) {
        cur_vma->vm_end = old_end;  // Nếu ánh xạ thất bại, hoàn nguyên
        free(area);
        return -1;
    }

    // Cập nhật kích thước đã mở rộng
    *inc_limit_ret = inc_amt;
    free(area);
    return 0;
}


    free(area);
    free(newrg);

    *inc_limit_ret = inc_amt;  // Trả về kích thước bộ nhớ đã thay đổi
    return 0;
}




src/mm-vm.c:599:5: warning: data definition has no type or storage class
  599 |     free(area);
      |     ^~~~
src/mm-vm.c:599:5: warning: type defaults to ‘int’ in declaration of ‘free’ [-Wimplicit-int]
src/mm-vm.c:599:5: warning: parameter names (without types) in function declaration
src/mm-vm.c:599:5: error: conflicting types for ‘free’; have ‘int()’
In file included from src/mm-vm.c:9:
/usr/include/stdlib.h:687:13: note: previous declaration of ‘free’ with type ‘void(void *)’
  687 | extern void free (void *__ptr) __THROW;
      |             ^~~~
src/mm-vm.c:600:5: warning: data definition has no type or storage class
  600 |     free(newrg);
      |     ^~~~
src/mm-vm.c:600:5: warning: type defaults to ‘int’ in declaration of ‘free’ [-Wimplicit-int]
src/mm-vm.c:600:5: warning: parameter names (without types) in function declaration
src/mm-vm.c:600:5: error: conflicting types for ‘free’; have ‘int()’
/usr/include/stdlib.h:687:13: note: previous declaration of ‘free’ with type ‘void(void *)’
  687 | extern void free (void *__ptr) __THROW;
      |             ^~~~
src/mm-vm.c:602:5: warning: data definition has no type or storage class
  602 |     *inc_limit_ret = inc_amt;  // Trả về kích thước bộ nhớ đã thay đổi
      |     ^
src/mm-vm.c:602:6: warning: type defaults to ‘int’ in declaration of ‘inc_limit_ret’ [-Wimplicit-int]
  602 |     *inc_limit_ret = inc_amt;  // Trả về kích thước bộ nhớ đã thay đổi
      |      ^~~~~~~~~~~~~
src/mm-vm.c:602:22: error: ‘inc_amt’ undeclared here (not in a function); did you mean ‘ino_t’?
  602 |     *inc_limit_ret = inc_amt;  // Trả về kích thước bộ nhớ đã thay đổi
      |                      ^~~~~~~
      |                      ino_t
src/mm-vm.c:603:5: error: expected identifier or ‘(’ before ‘return’
  603 |     return 0;
      |     ^~~~~~
src/mm-vm.c:604:1: error: expected identifier or ‘(’ before ‘}’ token
  604 | }
      | ^
make: *** [Makefile:41: obj/mm-vm.o] Error 1
