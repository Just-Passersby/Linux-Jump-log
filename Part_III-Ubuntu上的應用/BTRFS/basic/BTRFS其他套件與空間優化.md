# BTRFS其他套件與空間優化
BTRFS除了本身可以開啟透明壓縮來換換取更大的空間以外，BTRFS也有其他套件可以優化空間和更好的追蹤佔用情況。

# 壓縮和參考率套件：compsize
如果你設定完`fstab`後想要讓原本在BTRFS磁區的檔案可以被壓縮，你可以跑`sudo btrfs filesystem defragment -r -v -czstd $DIR`來對其壓縮
> `-czstd`是壓縮指令，這個參數必須要有，但這整行指令是碎片重組，會強行破壞現有的參考和快照。<br/>
> 使用SSD時，只有你從沒有透明壓縮變成檔案要有透明壓縮才跑一次就好，之後就不要再跑了

壓縮完後，你也不知道有沒有真的壓縮對吧？這時候就需要套件協助，BTRFS預設用`compsize`這個指令查詢，需要安裝套件，指令如下：
```bash
# APT
sudo apt install btrfs-compsize
```
如果`btrfs-compsize`找不到，直接找`compsize`也可以試試看。

使用方式：
```bash
# 需要Root權限
sudo compsize $PATH
```
> 注意，如果用`/`它會直接遞迴查詢，一旦查到`/boot`或者其他非BTRFS磁區就會失效\
> 實際範例：`/boot/efi/EFI/ubuntu/grubx64.efi: Not btrfs (or SEARCH_V2 unsupported).`

如果要希望避免查詢root遇到EFI分區然後查詢失敗，可以用`-x`把搜尋範圍限定在單個檔案系統上，這樣就不會查到根目錄底下的其他檔案磁區和檔案系統了。\
比如`sudo compsize -x /`

輸出參考：
```bash
$ sudo compsize /disks/FURY_Renegade_2T/SteamLibrary/
Processed 288832 files, 572989 regular extents (579843 refs), 56133 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       95%      492G         513G         515G       
none       100%      485G         485G         486G       
zstd        25%      7.2G          28G          28G       
prealloc   100%       38M          38M          38M    
```
你可以發現`Type`從左到右，從上到下，我先解釋第一排從左到右好了
- `Perc`: `Disk Usage`對比`Referenced`的總共佔用比例。
- `Disk Usage`: 搜索的目錄上的檔案在硬碟上的佔用大小。
- `Uncompressed`: 檔案沒被壓縮時的大小。
- `Referenced`: 目錄中沒有使用壓縮和參考的實際大小。

所以`Perc`的數值越低通常代表越省空間，但具體怎麼省還要看`Type`由上到下的數值
- `TOTAL`: 把底下所有計算方式和是否有用參考的方式都統計在這裡。
- `none`: 為被壓縮的檔案的佔用
- 壓縮法: 以我的使用環境中只有`zstd`這個欄位，代表透過`zstd`壓縮後的檔案大小
- `prealloc`: 預先分配出去的大小，通常可以忽略不計

這樣你就可以利用`compsize`來確定檔案具體是怎麼把空間省出來的了，但是只是有了壓縮有時還不能看出BTRFS省空間的威力，這時就要請出去重工具。

# 去除重複工具：Duperemove
去重也是BTRFS節省空間的方式之一，原理是Kernel有一個`ioctl`，可以讓去重工具基於這個函式去實現對檔案的掃描和比對，有機會我在展開詳細的原理。

那BTRFS目前有2個去重工具，一個是BEES，另一個是Duperemove，前者在NAS這類以儲存作為主要服務領域的情況很適合，後者則更靈活且適合我們桌面端。\
你問我為甚麼？因為BEES要維護Hash Table，Duperemove只要去指定好要去重的目錄就會幫忙去重，這篇展開來也可以說不少東西。

那Duperemove的安裝方式如下：
```bash
sudo apt install duperemove
```
裝好之後，就可以來用了，如果你想更直觀的比較，除了可以用上面的`compsize`，我們還可以用這些指令來比對佔用：
- `btrfs filesystem df $DIR`: 可以看Subvolume的佔用狀況，如果指定的參數不是子卷就不能檢查，使用範例：
  ```bash
  $ sudo btrfs filesystem df /disks/FURY_Renegade_2T/SteamLibrary 
  Data, single: total=976.01GiB, used=968.13GiB
  System, DUP: total=32.00MiB, used=160.00KiB
  Metadata, DUP: total=2.00GiB, used=1.69GiB
  GlobalReserve, single: total=512.00MiB, used=0.00B
  ```
- `btrfs filesystem usage $DIR`: 比`df`更細的佔用查看\
  不只Data、Metadata和System會分開列出，甚至還可以看到allocated和used的差異\
  理想上去重完會看到Data佔用降低，但Metadata佔用提昇，因為引用次數被增加

  使用範例：
  ```bash
  $ sudo btrfs filesystem usage /disks/FURY_Renegade_2T/SteamLibrary 
  Overall:
      Device size:                   1.82TiB
      Device allocated:            980.07GiB
      Device unallocated:          882.95GiB
      Device missing:                  0.00B
      Device slack:                    0.00B
      Used:                        971.51GiB
      Free (estimated):            890.82GiB      (min: 449.35GiB)
      Free (statfs, df):           890.82GiB
      Data ratio:                       1.00
      Metadata ratio:                   2.00
      Global reserve:              512.00MiB      (used: 0.00B)
      Multiple profiles:                  no
  
  Data,single: Size:976.01GiB, Used:968.13GiB (99.19%)
     /dev/nvme1n1p1        976.01GiB
  
  Metadata,DUP: Size:2.00GiB, Used:1.69GiB (84.47%)
     /dev/nvme1n1p1          4.00GiB
  
  System,DUP: Size:32.00MiB, Used:160.00KiB (0.49%)
     /dev/nvme1n1p1         64.00MiB
  
  Unallocated:
     /dev/nvme1n1p1        882.95GiB
  ```
- `filefrag -v $FILE`: 可以用來查看檔案的實際佔用位址，用來比對2個檔案是共用extent，輸出範例：
  ```bash
  $ filefrag -v Downloads/20260106_Patch.rar 
  Filesystem type is: 9123683e
  File size of Downloads/20260106_Patch.rar is 4556489244 (1112425 blocks of 4096 bytes)
   ext:     logical_offset:        physical_offset: length:   expected: flags:
     0:        0..   10965:  111474951.. 111485916:  10966:             shared
     1:    10966..   10966:  107841888.. 107841888:      1:  111485917: shared
     2:    10967..  261322:  111959296.. 112209651: 250356:  107841889: shared
     3:   261323..  490698:  112242326.. 112471701: 229376:  112209652: shared
     4:   490699..  973197:  112483584.. 112966082: 482499:  112471702: shared
     5:   973198.. 1005965:  112966084.. 112998851:  32768:  112966083: shared
     6:  1005966.. 1104269:  113007872.. 113106175:  98304:  112998852: shared
     7:  1104270.. 1112424:  111485918.. 111494072:   8155:  113106176: last,shared,eof
  Downloads/20260106_Patch.rar: 8 extents found
  ```
> 可選：`btrfs inspect-internal dump-tree $DISK`: 對BTRFS非常有興趣的，可以用這個指令觀察Kernel怎麼處理B-Tree的，這裡不展開

順帶一提，上面的`compsize`也是我的Steam遊戲庫，一模一樣的路徑，等等去重完會一起拿來比較。

接下來就可以來說duperemove怎麼用，詳細用法可以看[Docs](https://markfasheh.github.io/duperemove/duperemove.html)，這裡列舉常用的參數：
- `-r`: 遞迴執行。
- `-d`: 真的執行去重，如果沒有加上這個參數只會跑模擬並列出潛在候選。
- `--hashfile=$DEST`: 建立SQLite DB，會存放計算過得檔案的hash、mtime、大小等參數。

如果你只希望去重一次，或者偶爾去重，那可以考慮不存hashfile；如果你希望定期去重保證空間縮小，那使用`--hashfile`建立DB可以讓duperemove作到類似BEES的效果，還不會常駐佔用RAM。\
如果你有用Database，需要注意的是盡量避免讓duperemove去掃描到cache這類目錄，這種檔案TTL不高的地方多掃幾次後hashfile就髒掉了。

那Steam遊戲庫通常遊戲會一直更新，所以很適合用hashfile這個模式。\
在duperemove之前，可以先Snapshot，確保後續如果你去重完之後如果不符合預期或者不小心對不該CoW的檔案有操作後可以還原，當然如果不Snapshot也可以，就是要注意一點，但是Snapshot後做duperemove硬碟空間無法得到釋放，除非你之後把Snapshot刪掉，這點要注意。

那在去重之前，你可以不加`-d`來先做一次預測，以我的使用情境大概如下：
```bash
# 我有預先建立好放hashfile目錄
$ duperemove -r --hashfile=~/_duperemove_db/FURY_SteamLib.db /disks/FURY_Renegade_2T/SteamLibrary/
```
跑完之後duperemove會輸出可以被去重的檔案，我這裡抽個2個範例：
```bash
Showing 3 identical extents of length 84381696 with id e672d674
Start           Filename
0       "/disks/FURY_Renegade_2T/SteamLibrary/steamapps/common/Winter Memories/nw.dll"
0       "/disks/FURY_Renegade_2T/SteamLibrary/steamapps/common/Summer Memories/nw.dll"
0       "/disks/FURY_Renegade_2T/SteamLibrary/steamapps/common/Devil Bartender/nw.dll"
Showing 2 identical extents of length 111419472 with id 1bbf87ba
Start           Filename
1048576 "/disks/FURY_Renegade_2T/SteamLibrary/steamapps/common/Astral Party/8vJXn6CN/AstralParty_CN_Data/StreamingAssets/aa/StandaloneWindows64/1b93120ae60829d2c415fd1ea56e7303.bundle"
1048576 "/disks/FURY_Renegade_2T/SteamLibrary/steamapps/common/Astral Party/8vJXnINT/AstralParty_INT_Data/StreamingAssets/aa/StandaloneWindows64/22f7082a673a5a8182dbfbc8cb66049d.bundle"
```
結果預覽會顯示有幾個檔案(N identical)和每份去重區塊大小(length $BYTES，單位是Bytes，沒有K、M等)。\
對，我這裡用區塊(extent)，你可以看到下面的結果Start不是0，代表它去重的地方不是從檔起點開始，好了之後你想大致分析你可以丟給AI分析，以Claude來說，他會大概分析幾個比較大的結果，我的案例Claude Sonnet 4.6回應4項比較高價值的結果就節省了超過400MB，至少省下了接近0.5G，而且很多沒計算。

如果想知道具體怎麼算，算法是`length * (identical - 1)`，這樣就是單個項目的節省大小，因為去重是把重複的變成1個共用，所以identical那邊計算時要減1。

沒問題後，把上面的參數加上`-d`就會真的去重了，如果有hashfile，那會快很多，指令如下：
```bash
$ duperemove -dr --hashfile=~/_duperemove_db/FURY_SteamLib.db /disks/FURY_Renegade_2T/SteamLibrary/
```
這時候有database不用掃描真的快很多，那duperemove去重完後會跟你輸出結果，用一行字，結果如下：
```bash
Comparison of extent info shows a net change in shared extents of: 881917076
```
單位一樣是Bytes，除完後大概縮小了841 MB。\
`net change`翻譯是淨改變的意思，代表連去重後增加的Metadata大小都有算進去，所以是實打實的節省大小。
> Note: 這個預估結果僅建立在你所指定的duperemove範圍內，如果你去重前有複製裡面的檔案到其他地方那就不會被真的釋放，同時duperemove也評估不到。

接下來我就拿`compsize`和btrfs-progs的`df`和`usage`來驗證和比較吧：
```bash
...
Comparison of extent info shows a net change in shared extents of: 881917076
user@user-X870-Elite-Ice:~$ sudo compsize /disks/FURY_Renegade_2T/SteamLibrary/
Processed 288832 files, 528035 regular extents (579953 refs), 56133 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       96%      484G         502G         515G       
none       100%      478G         478G         486G       
zstd        25%      6.0G          24G          28G       
prealloc   100%       38M          38M          38M       
user@user-X870-Elite-Ice:~$ sudo btrfs filesystem df /disks/FURY_Renegade_2T/SteamLibrary 
Data, single: total=976.01GiB, used=964.71GiB
System, DUP: total=32.00MiB, used=160.00KiB
Metadata, DUP: total=2.00GiB, used=1.71GiB
GlobalReserve, single: total=512.00MiB, used=0.00B
user@user-X870-Elite-Ice:~$ sudo btrfs filesystem usage /disks/FURY_Renegade_2T/SteamLibrary 
Overall:
    Device size:                   1.82TiB
    Device allocated:            980.07GiB
    Device unallocated:          882.95GiB
    Device missing:                  0.00B
    Device slack:                    0.00B
    Used:                        968.14GiB
    Free (estimated):            894.24GiB      (min: 452.77GiB)
    Free (statfs, df):           894.24GiB
    Data ratio:                       1.00
    Metadata ratio:                   2.00
    Global reserve:              512.00MiB      (used: 0.00B)
    Multiple profiles:                  no

Data,single: Size:976.01GiB, Used:964.71GiB (98.84%)
   /dev/nvme0n1p1        976.01GiB

Metadata,DUP: Size:2.00GiB, Used:1.71GiB (85.66%)
   /dev/nvme0n1p1          4.00GiB

System,DUP: Size:32.00MiB, Used:160.00KiB (0.49%)
   /dev/nvme0n1p1         64.00MiB

Unallocated:
   /dev/nvme0n1p1        882.95GiB
```
恩...去重後評估是少800多MB，實際上直接少8G (WTF)，如果算是zstd的壓縮我在Steam遊戲庫上面合計省了31G，以515GB邏輯大小來說大約6%，看起來杯水車薪，但把視角拉回2020年遊戲大小膨脹前的時代，我可以多裝一個2020年前的3A，或者安裝約30款Galgame，甚至安裝一大堆DLSite下載的同人遊戲(大小普遍都在1G左右，少數大於1.5G，多數小於1G)，挺不錯的。

# 在/var中有趣的觀察
這個是無聊亂翻`/var`發現的，`/var`目錄因為基本上以存放log居多，所以天身很適合透明壓縮和去重施展，先給大家看看純透明壓縮有多恐怖：
```bash
user@user-X870-Elite-Ice:~$ sudo compsize /var
Processed 160804 files, 229504 regular extents (327938 refs), 66643 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       40%       13G          32G          47G       
none       100%       10G          10G          19G       
zstd        11%      2.5G          22G          27G       
prealloc   100%      2.5M         2.5M          36M       
user@user-X870-Elite-Ice:~$ sudo compsize /var/log
Processed 509 files, 155411 regular extents (155929 refs), 44 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL        9%      1.7G          18G          18G       
none       100%      660M         660M         642M       
zstd         6%      1.0G          17G          17G   
```
有了`compsize`就能直觀看到這種恐怖的數據，除此之外這裡還有個有趣的事情。

我們都知道Linux log有個logrotate的東西，他會定時幫忙切斷log然後壓縮成gzip，但是gzip在壓縮這類純文字log時壓縮率跟zstd差不多，但是壓縮速度慢很多。\
我們先來模擬一下logrotate會怎麼處理截斷的log檔案：讀取raw log -> gzip 壓縮寫入成壓縮檔 -> 刪除raw log\
看出來了嗎？3次I/O阿，而且CPU還要花比zstd更多的算力來壓縮，我們本來就有zstd了，該怎麼做？

首先，把logrotate的定時壓縮關閉，這樣就可以讓壓縮這件事回歸到FS上了，避免寫完gzip還要讓OS嘗試壓縮一次。\
再來，你可以近一步用duperemove做去重，這樣可以進一部省下空間，你說檔案重複機率高不高？你自己看看boot的log，就說重複率高不高：
```bash
user@user-X870-Elite-Ice:~$ sudo tail /var/log/boot.log.1
         Starting tailscaled.service - Tailscale node agent...
[  OK  ] Started unattended-upgrades.service - Unattended Upgrades Shutdown.
[  OK  ] Finished systemd-user-sessions.service - Permit User Sessions.
         Starting plymouth-quit.service - Terminate Plymouth Boot Screen...
         Starting setvtrgb.service - Set console scheme...
[  OK  ] Finished setvtrgb.service - Set console scheme.
[  OK  ] Created slice system-getty.slice - Slice /system/getty.
[  OK  ] Finished libvirt-guests.service - libvirt guests suspend/resume service.
[  OK  ] Started cups.service - CUPS Scheduler.
[  OK  ] Started libvirtd.service - libvirt legacy monolithic daemon.
user@user-X870-Elite-Ice:~$ sudo tail /var/log/boot.log.2
         Starting systemd-user-sessions.service - Permit User Sessions...
         Starting tailscaled.service - Tailscale node agent...
[  OK  ] Started unattended-upgrades.service - Unattended Upgrades Shutdown.
[  OK  ] Finished systemd-user-sessions.service - Permit User Sessions.
         Starting plymouth-quit.service - Terminate Plymouth Boot Screen...
         Starting setvtrgb.service - Set console scheme...
[  OK  ] Finished setvtrgb.service - Set console scheme.
[  OK  ] Created slice system-getty.slice - Slice /system/getty.
[  OK  ] Finished libvirt-guests.service - libvirt guests suspend/resume service.
[  OK  ] Started cups.service - CUPS Scheduler.
```
如果你真的要動，全域設定是`/etc/logrotate.conf`，但這裡預設`compress`是關閉的，你應該要去`/etc/logrotate.d/`底下把每個套件的設定檔看一遍，比如`/etc/logrotate.d/apt`：
```bash
/var/log/apt/term.log {
  rotate 12
  monthly
  compress      # 這裡就有開啟壓縮
  missingok
  notifempty
}

/var/log/apt/history.log {
  rotate 12
  monthly
  compress      # 這裡就有開啟壓縮
  missingok
  notifempty
}
```

剩下的就有興趣就各位自己實做吧。

# Reference
- [compsize GitHub Repo](https://github.com/kilobyte/compsize)
- [BTRFS Docs - Deduplication](https://btrfs.readthedocs.io/en/latest/Deduplication.html)
- [Duperemove GitHub Repo](https://github.com/markfasheh/duperemove)
- [Duperemove Docs](https://markfasheh.github.io/duperemove/duperemove.html)
