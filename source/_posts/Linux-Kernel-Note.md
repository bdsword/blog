title: Linux-Kernel-Note
date: 2015-12-25 11:23:36
tags:
- linux
- kernel
---

Linux Kernel Trace Notes
========================

> 這邊紀錄一些 Linux Kernel 課程研讀中的筆記

## MMU
1. __pgd_none__ 與 __pgd_present__ 是兩個剛好相反意思的 function
2. __pte_offset_kernel__ 僅用於 Kernel Space 的 Page 查找(是否意謂User Space有自己的分頁表位於HIGHMEM Zone中？！)
