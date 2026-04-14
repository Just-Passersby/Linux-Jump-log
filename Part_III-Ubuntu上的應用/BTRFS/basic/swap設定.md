# Swap file設定
我必須得說，雖然32G RAM不算小，但考量到部份程式其實十分依賴甚至強制要有Swap (比如VMware Workstation)，Swap必須要好好設定，安裝時Kubuntu遇上只給512MB Swap，基本等於沒給...所以必須好好設定一番。

## Swap大小


## BTRFS Swapfile建立


# Virtual Memory(VM)介入時機調整


# Reference
- [BTRFS Docs - Swapfile](https://btrfs.readthedocs.io/en/latest/Swapfile.html)
- [Red Hat Docs - Chapter 15. Swap Space](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-swapspace)
- [Smater Tools - Swap File Size Recommendations for Linux SmarterMail Installations](https://portal.smartertools.com/kb/a3754/swap-file-size-recommendations-for-linux-smartermail-installations.aspx)
- [Red Hat Docs - Chapter 13. Getting started with swap](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/getting-started-with-swap)
- [Tsung's Blog - Linux 設定 vm.swappiness 調整 SWAP 使用時機](https://blog.longwin.com.tw/2024/05/linux-set-swap-ram-memory-usage-2024/)
- [farseerfc - 【譯】替 swap 辯護：常見的誤解 ](https://farseerfc.me/in-defence-of-swap.html)