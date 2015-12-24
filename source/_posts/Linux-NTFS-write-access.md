title: Linux NTFS write access.
date: 2014-09-28 22:31:10
tags:
- linux
- kernel
---

Linux NTFS write access
=======================

> 壞人因畏懼而服從，好人因愛而服從 — 亞里士多德

在使用Funtoo Linux的時候，喜歡採用自己調整、編譯的Kernel。

在File Systems的部份，總是只勾選幾項重點項目：

1. Ext系列是必備的
2. JFS, XFS採用Module的方式
3. DOS/FAT/NT Filesystems，則是除了NTFS debugging support都勾選了

原本以為這樣就可以擁有NTFS的寫入存取，沒想到在一次使用 [Easy2Boot](http://www.easy2boot.com/) 軟體的時候，遇到了點障礙。用Easy2Boot製作好的隨身碟，想放入iso檔案，在Windows環境下沒問題，但在Funtoo底下，卻沒有對NTFS的寫入權，後來Google之後得知，需要安裝ntfs-3g軟件(不確定是否必要)，安裝之後發現Kernel沒有開啟必要的Fuse的功能，將該選項加入之後，重編核心，終於能順利寫入NTFS了。
