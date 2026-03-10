# Snapshot實做與原理
## BTRFS建立快照的原理
雖然我是為了Snapshot而學BTRFS，這件事對於我和大多數人而言是個「功能」(比如風滾草的Snapper建立在此之上)，但從BTRFS的角度而言，Snapshot是個「動作」跟「動詞」，甚麼意思？

可以看看官方[Docs](https://btrfs.readthedocs.io/en/latest/Subvolumes.html#nested-subvolumes)，Snapshot僅有在Subvolume章節的Nested subvolumes部份出現Snapshot的指令，至於BTRFS manual page，也是把Snapshot相關操作放在`man btrfs-subvolume`裡面。

但從官方文件並不能說明為甚麼要把Snapshot放到Subvolume裡面，如果說官方文件是基於設計和理論寫出來的，我們可以推導出這個結論：Snapshot**等於**Subvolume！欸不是，你是不是在說傻話？其實不然，因為BTRFS如同它的名字**B-Tree File System**，所有東西都是B-Tree，無論是Storage Pool、每一筆資料還是Metadata都是以B-Tree的形式儲存，有關於BTRFS的Snapshot的理論細節[Farseerfc的文章](https://farseerfc.me/btrfs-vs-zfs-difference-in-implementing-snapshots.html)寫的很好，這篇文章也能說明為甚麼Snapshot以官方的角度而言，是歸類在Subvolume而不是獨立的Features，同時也說明為甚麼Root Subvolume ID是5，至於256才是該磁區第一個被指令建立的SubVolume的ID(中間ID一路保留到255)。

> 在此，無論你有沒有看過或學過ZFS，BTRFS雖然與ZFS都有參考WAFL和CoW，但你必須知道，BTRFS是**借鑒**可以寫到無論何處的理念，不單單想成為Linux正統的CoW FS，而是充滿野心的大膽嘗試，跟Linux特性的深度結合，成為像是瑞士刀與變形金剛存在，絕對不是ZFS的拙劣模仿；而ZFS則是透過SDS(軟體定義儲存)對WAFL的最強模仿，甚至在某方面青出於藍。在開源領域，BTRFS與ZFS雖然也有持續互相借鑒，但核心理念與實現邏輯完全不同，雖然兩者唯二的共通點就是基於CoW和都能取代傳統RAID卡，但BTRFS相比ZFS更加符合「File System」這個描述。

## BTRFS新增和使用Snapshot
正如前面所說，Snapshot是被歸類在Subvolume裡面，所以你要看Snapshot要怎麼用要輸入`sudo btrfs subvolume`才能看到怎麼用：
```bash
...

        List subvolumes and snapshots in the filesystem.
    btrfs subvolume snapshot [-r] [-i <qgroupid>] <subvolume> { <subdir>/<name> | <subdir> }

...
```
> 補充：
> - `-r`: 是建立唯讀快照的意思，不加`-r`代表這個快照可以隨意修改。這是可選項。
> - `<subvolume>`: 這裡並不是`btrfs subvolume list`的子卷名稱，而是你要Snapshot的目錄位置，該目錄必須有掛載subvolume。
> - `{ <subdir>/<name> | <subdir> }`: `subdir`指的是你要放snapshot的位置，如果你想要指定位置後同時改名可以在`subdir`後面用斜線`/`分隔並命名該Snapshot的名字。

我就以我的遊戲硬碟為例，相關設定環境請參考[BTRFS使用原則與系統fstab設定](../basic/BTRFS使用原則與系統fstab設定.md)，我等等會對Steam遊戲庫操作快照，這裡使用`compsize`觀察空間佔用變化，可以參考[BTRFS其他套件與空間優化](../basic/BTRFS其他套件與空間優化.md)如何安裝與使用。

這是我目前這顆硬碟快照前的狀態：
```bash
$ sudo btrfs subvolume list .
ID 256 gen 45411 top level 5 path @FURY_2T
ID 258 gen 43823 top level 5 path @Hgames
ID 259 gen 45410 top level 5 path @SteamGames
ID 261 gen 23690 top level 5 path @No_CoW

$ sudo compsize .
Processed 352040 files, 1791132 regular extents (1797938 refs), 60858 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       91%      773G         841G         842G       
none       100%      713G         713G         714G       
zstd        46%       59G         127G         128G       
prealloc   100%       20M          20M          20M    
```
接下來我替我的Steam目錄做快照，可以看到BTRFS具體Snapshot會怎麼影響佔用：
```bash
# Steam佔用：
$ sudo compsize ./SteamLibrary/
Processed 263722 files, 494734 regular extents (501538 refs), 52858 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       95%      454G         473G         475G       
none       100%      448G         448G         449G       
zstd        24%      6.1G          25G          25G       
prealloc   100%       20M          20M          20M   

# 建立Snapshot到/disk/FURY_Renegade_2T/_Snapshots (不改名)
$ sudo btrfs subvolume snapshot SteamLibrary/ _Snapshots/
Create a snapshot of 'SteamLibrary/' in '_Snapshots//SteamLibrary'

# 佔用檢查
$ sudo compsize .
Processed 615762 files, 1791132 regular extents (2299476 refs), 113716 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       91%      773G         841G         1.2T       
none       100%      713G         713G         1.1T       
zstd        46%       59G         128G         153G       
prealloc   100%       20M          20M          40M    
```
可以看到`TOTAL`的部份Disk Usage維持在773G，但是Referenced增加到1.2T，對比原本842G對的上增加整個Steam遊戲庫的大小，至於`prealloc`的部份那20M剛好都是Steam遊戲庫的，在Refernced可以看見快照整個遊戲庫後變成40M，但是實際佔用只有20M。

> 從BTRFS的角度而言，Snapshot是個「動作」跟「動詞」... 也就是說，Snapshot**等於**Subvolume！

喔！？那我們來驗證我前面說的正不正確吧：
```bash
$ sudo btrfs subvolume list .
ID 256 gen 45412 top level 5 path @FURY_2T
ID 258 gen 43823 top level 5 path @Hgames
ID 259 gen 45412 top level 5 path @SteamGames
ID 261 gen 23690 top level 5 path @No_CoW
ID 262 gen 45412 top level 256 path _Snapshots/SteamLibrary
```
沒有錯的，今天當我「快照」一個子卷，其實就是複製一個Subvolume的B-Tree節點，並且共用同一筆資料，而且這個子卷有一個獨立ID，連影響增量備份的`gen`參數都一模一樣(剛好沒有遊戲更新)。只是因為整個硬碟我透過`@FURY`掛載在(硬碟的)根目錄，所以該Snapshot的父節點變成`256`，對應的路徑就是`@FURY_2T/_Snapshot`。

因此在BTRFS中，子卷Subvolume是個**名詞**，快照Snapshot是個**動詞**(複製子卷)。

其實這個時候對於被快照的子卷自身的資訊也會有影響:
```bash
$ sudo btrfs subvolume show SteamLibrary/
@SteamGames
        Name:                   @SteamGames
        UUID:                   69aef626-2884-e641-8e64-efc0e8b0815b
        Parent UUID:            -
        Received UUID:          -
        Creation time:          2025-12-11 18:54:11 +0800
        Subvolume ID:           259
        Generation:             45415
        Gen at creation:        23176
        Parent ID:              5
        Top level ID:           5
        Flags:                  -
        Send transid:           0
        Send time:              2025-12-11 18:54:11 +0800
        Receive transid:        0
        Receive time:           -
        Snapshot(s):
                                @FURY_2T/_Snapshots/SteamLibrary
        Quota group:            n/a
```

雖然指令表面上看起來可以到處幫每個路徑建立Snapshot，但參數明確說到必須是`subvol`，所以我們來試試看沒有掛載子卷的目錄可不可以被Snapshot：
```bash
$ sudo btrfs subvolume snapshot Ubuntu-Budgie_Somefile/ _Snapshots/old-system-file
ERROR: Not a Btrfs subvolume: Invalid argument

$ sudo btrfs subvolume snapshot Ubuntu-Budgie_Somefile/ _Snapshots/
ERROR: Not a Btrfs subvolume: Invalid argument
```
沒錯，會跳提示說這不是子卷，不給我快照。

需要注意的是，如果你有用巢狀子卷，那在Snapshot時這些子卷內的子卷、並不會被跟著快照，而是留下ID 2的空目錄，你必須自己手動替每個子卷做快照，這在你用Snapper、TimeShift等快照工具也有這個問題，來看官方範例：
```bash
$ btrfs subvolume create subvol1
$ btrfs subvolume create subvol1/subvol2
$ btrfs subvolume snapshot subvol1 snap1
$ find -ls
121093  0  drwxr-xr-x  1  user  users    24  Jul 30  12:34  .
   256  0  drwxr-xr-x  1  user  users    14  Jul 30  12:34  ./subvol1
   256  0  drwxr-xr-x  1  user  users     0  Jul 30  12:34  ./subvol1/subvol2
   257  0  -rw-r--r--  1  user  users     0  Jul 30  12:34  ./subvol1/subvol2/file
   256  0  drwxr-xr-x  1  user  users    14  Jul 30  12:34  ./snap1
     2  0  drwxr-xr-x  1  user  users     0  Jul 30  12:34  ./snap1/subvol2
```

那我們再來建立一個唯讀子卷，看看跟一般子卷的差異：
```bash
$ sudo btrfs subvolume snapshot -r SteamLibrary/ _Snapshots/ReadOnly_Steam
Create a readonly snapshot of 'SteamLibrary/' in '_Snapshots/ReadOnly_Steam'

$ rm -rf _Snapshots/SteamLibrary/libraryfolder.vdf 
$ rm -rf _Snapshots/ReadOnly_Steam/libraryfolder.vdf 
rm: cannot remove '_Snapshots/ReadOnly_Steam/libraryfolder.vdf': Read-only file system
$ sudo rm -rf _Snapshots/ReadOnly_Steam/libraryfolder.vdf 
rm: cannot remove '_Snapshots/ReadOnly_Steam/libraryfolder.vdf': Read-only file system
```
凶悍的不行！欸～☝️🤓 我想起來，`lsattr`是可以看到檔案如果有`i`的標示該檔案即使root也只有唯讀的權限，必須透過`chattr`解除鎖定才可以修改，那我們來看看他的狀態吧：
```bash
$ lsattr _Snapshots/
---------------------- _Snapshots/SteamLibrary
---------------------- _Snapshots/ReadOnly_Steam
```
阿這...非常之凶狠阿，就算你會用`lsattr`你還是找不到問題...真正能看到差異的地方，就是用`btrfs subvolume show $SUBVOL`來看子卷狀態：
```bash
$ sudo btrfs subvolume show _Snapshots/SteamLibrary/
@FURY_2T/_Snapshots/SteamLibrary
        Name:                   SteamLibrary
        UUID:                   dad0b0ff-6f03-5b44-b630-543f7c394f04
        Parent UUID:            69aef626-2884-e641-8e64-efc0e8b0815b
        Received UUID:          -
        Creation time:          2026-03-08 00:13:08 +0800
        Subvolume ID:           262
        Generation:             45416
        Gen at creation:        45412
        Parent ID:              256
        Top level ID:           256
        Flags:                  -
        Send transid:           0
        Send time:              2026-03-08 00:13:08 +0800
        Receive transid:        0
        Receive time:           -
        Snapshot(s):
        Quota group:            n/a

# ===================================================================

$ sudo btrfs subvolume show _Snapshots/ReadOnly_Steam/
@FURY_2T/_Snapshots/ReadOnly_Steam
        Name:                   ReadOnly_Steam
        UUID:                   2a10d1c8-22fa-b044-bf93-92648ec13b28
        Parent UUID:            69aef626-2884-e641-8e64-efc0e8b0815b
        Received UUID:          -
        Creation time:          2026-03-08 01:07:59 +0800
        Subvolume ID:           263
        Generation:             45415
        Gen at creation:        45415
        Parent ID:              256
        Top level ID:           256
        Flags:                  readonly
        Send transid:           0
        Send time:              2026-03-08 01:07:59 +0800
        Receive transid:        0
        Receive time:           -
        Snapshot(s):
        Quota group:            n/a
```
看出來了嗎？`ReadOnly_Steam`的`Flags`欄位多了一個`readonly`，此時這個快照可以透過增量備份的方式傳輸到其他磁區了，這個我留到[子卷增量傳輸](./子卷增量傳輸.md)再說。

那此時這個快照就無敵或者我只是想修改快照後再傳輸就不行嗎？不！btrfs-progs有個工具叫做`property`，可以設定不同目錄和子卷的設定，比如該子卷可以用更高的壓縮比，是否唯讀也是可以設定的，直接上範例：
```bash
$ sudo btrfs property set _Snapshots/ReadOnly_Steam/ ro false

$ sudo btrfs subvolume show _Snapshots/ReadOnly_Steam/
@FURY_2T/_Snapshots/ReadOnly_Steam
        Name:                   ReadOnly_Steam
        UUID:                   2a10d1c8-22fa-b044-bf93-92648ec13b28
        Parent UUID:            69aef626-2884-e641-8e64-efc0e8b0815b
        Received UUID:          -
        Creation time:          2026-03-08 01:07:59 +0800
        Subvolume ID:           263
        Generation:             45415
        Gen at creation:        45415
        Parent ID:              256
        Top level ID:           256
        Flags:                  -
        Send transid:           0
        Send time:              2026-03-08 01:07:59 +0800
        Receive transid:        0
        Receive time:           -
        Snapshot(s):
        Quota group:            n/a

$ sudo rm -rf _Snapshots/ReadOnly_Steam/libraryfolder.vdf 
```
沒錯，一個子卷是否唯讀看`ro`是否開啟就好，所以你要打開就寫成`ro true`就好

> btrfs-progs是支持類似Cisco指令簡寫的，比如這裡`property`可以縮寫成`prop`，這在man page有展示出來。

雖然子卷可以像是一般的目錄一樣用`rm -rf`移除，但是碰到這種唯讀子卷就算你有root也沒用，只有你手握`btrfs-progs`時，即使快照/子卷唯讀，依舊可以透過`btrfs subvolume delete $SUBVOL`刪除：
```bash
$ sudo btrfs property set _Snapshots/ReadOnly_Steam/ ro true
$ rm -rf _Snapshots/ReadOnly_Steam/
rm: cannot remove '_Snapshots/ReadOnly_Steam/ ... ': Read-only file system

$ sudo btrfs subvolume delete _Snapshots/ReadOnly_Steam/
Delete subvolume 263 (no-commit): '/disks/FURY_Renegade_2T/_Snapshots/ReadOnly_Steam'
$ sudo btrfs subvolume list .
ID 256 gen 45415 top level 5 path @FURY_2T
ID 258 gen 43823 top level 5 path @Hgames
ID 259 gen 45415 top level 5 path @SteamGames
ID 261 gen 23690 top level 5 path @No_CoW
ID 262 gen 45416 top level 256 path _Snapshots/SteamLibrary
```
畢竟`rm -rf`是針對檔案操作，但是btrfs-progs是檔案系統層級的設定，Flags出現`readonly`是對於Coreutils的「降維打擊」，檔案系統的問題只能透過檔案系統的工具解決。

## 後話
我其實本來還想直接接TimeShift甚至是Snapper，但是仔細展開太長了，我放到另外2篇再來說說吧，我真沒想到Snapshot展開東西比我設定`fstab`那篇還多...
- TimeShift: [TimeShift_BTRFS模式安裝與設定](./TimeShift_BTRFS模式安裝與設定.md)
- Snapper: [Snapper安裝與設定](./Snapper安裝與設定.md)

# Reference
- [BTRFS Docs - Subvolumes](https://btrfs.readthedocs.io/en/latest/Subvolumes.html)
- [OOS Lab - Btrfs vs ZFS 快照實現原理深度比較｜CoW 檔案系統技術解析](https://www.osslab.com.tw/btrfs-vs-zfs-snapshot/)
- [Farseerfc - Btrfs vs ZFS 實現 snapshot 的差異](https://farseerfc.me/btrfs-vs-zfs-difference-in-implementing-snapshots.html)
- `man btrfs`
- `man btrfs-subvolume`
- `man btrfs-property` 
