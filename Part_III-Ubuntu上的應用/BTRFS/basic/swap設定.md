# Swap file設定
我必須得說，雖然32G RAM不算小，但考量到部份程式其實十分依賴甚至強制要有Swap (比如VMware Workstation)，Swap必須要好好設定，安裝時Kubuntu遇上只給512MB Swap，基本等於沒給...所以必須好好設定一番。

# Swapfile在BTRFS上需要注意的點
有人說BTRFS不能建立Swapfile (比如說Red Hat)，不算錯，CoW的天性是這樣，但是隨著BTRFS的更新並搭配一些設定，BTRFS要設定Swapfile不是問題，但是有很多細節需要注意。

## 建立Swapfile時的限制
- filesystem
  - 設備只能單一<br/>
    這裡的設備是從BTRFS角度看的，任何分區(比如`/dev/sda1`、`/dev/sda2`)都是一個block device，2種設備組合成一個pool都會讓Swapfile無法建立。<br/>
    反過來說，如果使用mdadm(`/dev/md0`)或者LVM(`/dev/vg/lv`)先組合好後再格式化成BTRFS，不管底下有幾個設備BTRFS都只會看到一個device。

    如果還是不懂，直接使用`sudo btrfs filesystem show`就可以看到BTRFS怎麼區分

  - data profile為*single*
- subvolume
- swapfile

## 存在Swapfile時對檔案系統操作的影響或限制
asdas

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