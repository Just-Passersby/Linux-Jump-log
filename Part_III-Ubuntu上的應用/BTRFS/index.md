# BTRFS

[toc]

# Introduction
BTRFS可以說是正統繼承ext4的下一代檔案系統，繼ext4之後同Linux Kernel使用GPL授權，受ZFS啟發，因為ZFS的授權協議(CDDL)與Linux Kernel的授權不相容且受限於Sun所以Oracle開發BTRFS與之抗衡，2者具備CoW特性，甚至能取代傳統的LVM和RAID，但隨著Sun被Oracle收購後BTRFS發展受阻，甚至一度被Linux社群（Red Hat帶頭）打入冷宮，接著被Meta (原Facebook)挖角原班開發者變成了一個成熟的檔案系統。

資源消耗上比ZFS節省很多，但是須注意Raid 5/6上使用BTRFS有Write hole(寫入黑洞)的問題，斷電資料會損毀。

潛在後繼者：[bcachefs](https://bcachefs.org/)

# fstab
- [BBTRFS使用原則與系統fstab設定](./basic/BTRFS使用原則與系統fstab設定.md)
- [swap設定](./basic/swap設定.md)