# BTRFS使用原則與系統fstab設定
主流的發行版雖然都支援BTRFS，但是Ubuntu體系對BTRFS其實並沒有優化配置，在Kubuntu 24.04.3 LTS (其他Ubuntu不確定會不會這樣)甚至會直接宣告`autodefrag`這個參數，造成寫入放大(Write Amplification)的問題，導致SSD壽命縮短，這是Kubuntu 24.04.3預設的`/etc/fstab`:
```bash
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a device; this may
# be used with UUID= as a more robust way to name devices that works even if
# disks are added and removed. See fstab(5).
#
# <file system>             <mount point>  <type>  <options>  <dump>  <pass>
UUID=7EA5-9093                            /boot/efi      vfat    defaults   0 2
UUID=b7d50a0d-5ccc-45f3-aeaf-4eb03768ba34 /              btrfs   subvol=/@,defaults,noatime,autodefrag 0 0
UUID=b7d50a0d-5ccc-45f3-aeaf-4eb03768ba34 /home          btrfs   subvol=/@home,defaults,noatime,autodefrag 0 0
UUID=b7d50a0d-5ccc-45f3-aeaf-4eb03768ba34 /swap          btrfs   subvol=/@swap,defaults 0 0
/swap/swapfile                            swap           swap    defaults   0 0
tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0
```

在Fedora和OpenSUSE中，不只預設使用BTRFS，還會有較適合發揮BTRFS優勢的設定，比如兩者系統預設開啟壓縮，並且都內建`Snapper`來管理快照，OpenSUSE甚至有著（據我所知）最優秀的BTRFS預設項，做到真正的「開箱即用」。

當你決定使用BTRFS作為你的主力檔案系統，決定參考以下設定的時候，記住，務必讓你的每個BTRFS磁區**一定要留15%可用空間**避免無法平衡和處理Metadata。

再來非企業環境只是個人使用就不要使用Quota (Qgroup)，BTRFS使用Qgroup會有很**嚴重**的性能問題。

# 根目錄fstab設定
Ubuntu雖然預設沒有把BTRFS的優勢功能都設定出來，甚至在Kubuntu或部份官方發行版上沒有任何快照工具（除了用`btrfs`來下快照），但Ubuntu預設的BTRFS布局是**扁平化布局**和使用`@`作為SubVolume開頭，這意味著要設定快照套件會容易很多，把適當的`/etc/fstab`參數設定好並搭配TimeShift可以做出比Fedora舒服的BTRFS體驗（Fedora預設的格式與快照工具Snapper整合度相對不高且設定較為麻煩）。

在設定快照之前，我建議先把`/etc/fstab`設定完成，因為BTRFS有內建檔案系統等級的壓縮，如果你希望設定完`/etc/fstab`就馬上全磁區壓縮那這會對硬碟造成大量刷寫，在有儲存快照的情況這個問題會加劇，那如果你是怕設定錯炸開我會更建議直接把`/etc/fstab`備份好，如果設定炸開導致根目錄掛不了也可以透過LiveCD把硬碟掛載還原備份救回來。

正如前面所說，Kubuntu預設參數不只不能發揮BTRFS的優勢，甚至對於SSD還很傷，比如`autodefrag`，它會在每一次有檔案更動就會進行重組，大檔案強連續，小檔案拼成同個metadata，這也是這個參數會增加寫入放大(Write Amplification)的原因。

BTRFS除了支援快照以外還有另一大殺手鐧就是資料壓縮，主流都會推薦使用Zstandard，因此設定BTRFS的時候一般都會開啟`compress=zstd`，Fedora預設等級是`zstd:1`，對於進幾年的CPU只要不是特別弱的或者太老的，比如有個i5 12th或Ryzen 5 3500X以上的建議可以開到`zstd:3`，這是壓縮率相對高且對CPU性能影響小的壓縮等級了，超過4以後邊際效應會放大並且如果每筆資料都要做高等級壓縮會變成整個系統就會變成CPU Bound，不只SSD的讀寫速度吃不滿，也會讓整個系統變卡，當然完全不在意空間消耗或者就是會怕CPU會有負擔影響遊戲性能之類的可不開或者只開`zstd:1`。

若沒指定壓縮等級，直接輸入`compress=zstd`預設使用3級壓縮。

> 歷史補充：
> 一開始BTRFS可以說幾乎是爛尾狀態，丟到BTRFS的資料容易消失或是有問題，隨著Red Hat把BTRFS從格式化選項中拔掉可以說基本被打入冷宮，直到Meta要在組織內部數大量Linux Server同時又需要CoW。隨著Meta下場加入開發，BTRFS從Gzip（壓縮率高但壓縮耗時）和LZo（壓縮與解壓縮非常快但壓縮率低）這些不太理想的選擇中出現了Zstandard (Zstd)這個壓縮率高且解壓速度快的壓縮法，並且Zstd在不同的壓縮等級下解壓速度還一致。

那這就是我改完`/etc/fstab`根目錄部份的設定的樣子，提供參考：
```bash
# <file system>             <mount point>  <type>  <options>  <dump>  <pass>
UUID=7EA5-9093                            /boot/efi      vfat    defaults   0 2
UUID=b7d50a0d-5ccc-45f3-aeaf-4eb03768ba34 /              btrfs   subvol=/@,defaults,noatime,compress=zstd:3,discard=async 0 0
UUID=b7d50a0d-5ccc-45f3-aeaf-4eb03768ba34 /home          btrfs   subvol=/@home,defaults,noatime,compress=zstd:3,discard=async 0 0
UUID=b7d50a0d-5ccc-45f3-aeaf-4eb03768ba34 /swap          btrfs   subvol=/@swap,defaults 0 0
/swap/swapfile                            swap           swap    defaults   0 0
tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0
```

那這裡補充一些參數：
- `noatime`: `atime`在Linux或者說Unix的世界裡代表上次存取時間「Access Time」，`noatime`代表關閉紀錄存取時間，在Ext4時期如果開著頂多只是幾位元最多幾個分區的刷寫，但在BTRFS這種具備CoW的File System，這會導致檔案系統每次一個檔案被存取都會建立一個時間戳記然後修改檔案指標，這不只導致檔案碎片化，也會導致資料不斷新增然後再回收，這就是導致SSD寫入放大的原因，宣告`noatime`可以有效避免這個問題，PC領域也幾乎不需要用到檔案上次被存取的時間，所以即使是Ext4也十分建議使用這個參數。
- `discard=async`: 這是BTRFS特有的功能，當檔案在系統上被刪除時系統會在被刪除的檔案上貼上類似「以刪除」的Tag或者是修改Metadata表示這塊區域沒有檔案佔用，傳統的`discard`相當於是宣告了`discard=sync`，這表示我檔案一被貼標籤或從清單上面抹除SSD就會馬上從系統上收到Trim指令把空間釋放獲得更大的預留空間，當刪除檔案數量一多尤其是小檔案就會變成I/O Bound場景，而使用`discard=async`則是會先在SSD中紀錄哪些檔案被丟棄了，等到被刪除的量且I/O負載輕鬆的時候再來處理，搭配`fstrim.timer`可以把檔案刪除對系統的影響大幅優化。有個Oracle的[部落格文章](https://blogs.oracle.com/linux/btrfs-discard)說明了BTRFS在非同步狀態下的回收機制判斷，有興趣可以參考。
- `compress=zstd:3`: 如果你覺得CPU算力不重要，硬碟空間更重要，除了可以提高壓縮等級以外，你也可以使用`compress-force=zstd:$LEVEL`讓BTRFS必須嘗試壓縮，只有壓縮的結果幾乎沒變甚至變大才存回原始檔。

# 虛擬機、Database和Swapfile的檔案屬性
由於BTRFS的CoW特性，虛擬機和Database這種每次操作都會造成大量小寫入與修改的情況下因為舊資料也不能馬上刪除，所以就會變成不斷分配位置然後再回收，為了避免CoW特性導致在SSD操作這些檔案時性能大幅下降，透過給目錄新增`C`這個Tag那後續移至或新增至這個目錄的檔案都會帶有`C`這個Tag，`C`這個代表**關閉CoW**，這樣可以避免虛擬機、Database會一直瘋狂寫入新資料，而是只要寫入到這個目錄內的話檔案的處理方式類似Ext4，檔案需要修改就是直接刷寫該檔案的區塊，只是從Ext4那種日誌型紀錄變成依舊有CoW的Metadata，但檔案本體沒CoW。

那使用BTRFS一般來說除了上述虛擬機(`.qcow2`、`.vmx`)、Database和Swapfile這種類型的檔案平日其實不建議把CoW關閉，因為關閉CoW會讓檔案的儲存行為類似Ext4，這就會導致失去Checksum避免位元腐爛、無法壓縮檔案，這兩個功能都依賴CoW才能正常運作，雖然此時快照功能也還能用，但是如果對於關閉CoW的目錄快照其實BTRFS會強行做一次CoW，這也會讓無CoW目錄的抗碎片能力會大幅下降。

那來實驗一下，如果一個目錄被賦予了No Cow的Tag，那檔案複製過去會怎麼樣

```bash
$ lsattr . | grep _No_CoW
---------------C------ ./_No_CoW
$ lsattr . | grep bookmarks_2025_11_30.html
---------------------- ./bookmarks_2025_11_30.html
$ cp --reflink=always bookmarks_2025_11_30.html _No_CoW/
cp: failed to clone '_No_CoW/bookmarks_2025_11_30.html' from 'bookmarks_2025_11_30.html': Invalid argument
$ cp bookmarks_2025_11_30.html _No_CoW/
$ lsattr _No_CoW/
---------------C------ _No_CoW/bookmarks_2025_11_30.html
```
可以看到我第一次`cp`我使用了`--reflink=always`，在BTRFS的磁區使用這個參數表示複製檔案都只建立新的Metadata但指標但是都指向同一筆資料，但No CoW的目錄和檔案只支持原地複寫，所以無論Source還是Destination是No CoW檔案使用`alaways`就會報錯，那在使用BTRFS的情況下預設是`--reflink=auto`，如果在2者都在CoW目錄會使用`reflink`，若至少其中一邊是No CoW檔案或目錄，就會使用深度複製(Deep Copy，在硬碟複製整份檔案的資料)，如果想要複製時資料也被完全複製可以使用`--reflink=never`來實現無論目的地都用Deep Copy。

# 非系統BTRFS硬碟掛載
如果今天像我是把一個SSD格式化成BTRFS後可能平時是Linux和Windows兩邊互相掛載共用，那就不會考慮把硬碟混合到同一個Part pool，但如果真的要混合成一個Pool跟LVM一樣，會比較建議直接用RAID 0換取更多性能或RAID 1來保障資料完整性，尤其是有RAID 1 (包括10、01)甚至能讓BTRFS檢測到位元腐爛後自動修復檔案。

Linux是三作業系統中「真正的」屬於自己的作業系統，硬碟你想掛那都沒差（掛在不該掛的地方如果指令擋不住自己負責），但一般來說如果你是FHS流派那會把硬碟掛載在`/mnt`底下，但是按照FHS標準`/mnt`是用於臨時掛載分區的，所以也有人會選擇直接放在根目錄，我則是用比較類似MacOS的布局，我會在根目錄建立`/disks`用來放要常駐掛載的磁區，那我用`FURY_Renegade_2T`這個名字表示這個要掛載的磁區。

那在直接寫`fstab`之前，你可以透過GUI或指令工具做分區然後格式化，那以我的例子是格式化後就直接放著，要用就透過檔案總管手動掛載，資料和資料夾直接放在硬碟根目錄，那為了深度優化BTRFS以及強化特性，我建議替這個硬碟先做一點設定：設定子卷（SubVolume）。BTRFS設定SubVolume可以切分快照區域，SubVolume有2種布局方式：平鋪式和嵌入式，前者是把SubVolume放在分區根目錄，然後透過掛載的方式，讓你可以在你期望的位置訪問這些子卷，如果你有仔細看系統分區的掛載你可以發現他同時掛載了`/`、`/home`和`/swap`，然後他們的SubVolume分別在`/@`、`/@home`和`/@swap`並透過`subvol=$PATH`來指定子卷，`subvol`底下的`/`代表的是這個磁區的根目錄，而不是整個系統的根目錄，所以不同硬碟不同磁區的子卷是不互通的，有點像是Windows那種分區不同就有不同編號那種邏輯，所以可以看到系統分區預設的3個SubVolume都在最頂端，也就是分區根目錄底下，這就是平鋪式的布局，至於這些子卷為何開頭要加一個`@`，主要是為了方便識別這是子卷，子卷被建立後看起來跟一般的目錄沒有區別，為了方便識別這種重要的目錄尤其是要拿來掛的都會在根目錄建立子卷並且名字帶有`@`。

嵌入式子卷也可以稱為巢狀子卷，也就是在子卷底下又有一個子卷，那平鋪式和嵌入式可以共存，但需要注意的是如果巢狀子卷是在快照後才建立的，那被快照還原後這個子卷會呈現一種存在但不知道該掛載在那的地方，需要自己去想辦法撈回來。以Kubuntu 24.04 LTS為例，除了`/etc/fstab`上的3個平鋪在根目錄上面的3個子卷，系統根目錄`@`底下又有額外2個子卷：
```bash
$ sudo btrfs subvolume list /
ID 256 gen 17275 top level 5 path @
ID 257 gen 17275 top level 5 path @home
ID 258 gen 15615 top level 5 path @swap
ID 259 gen 35 top level 256 path var/lib/portables
ID 260 gen 35 top level 256 path var/lib/machines
```
可以看到像是系統子卷`@`節點ID是256，他的上層ID是5，5就是整個BTRFS分區的根目錄。

你說這兩種布局方式誰比較好？當然是沒有最好的方式，只有最適合「需求」的方式，比如這個目錄你要考慮快照、檔案切分，那使用平鋪式的放在根目錄會更優；如果只是要把Snapshot紀錄的檔案切開，那直接用成巢狀會更簡單

需要注意的是這些子卷可以被`rm -rf`刪除，所以進行操作的時候還請注意...我測試的時候就不小心把比較不重要的系統子卷刪除了，還好可以馬上加回來...

那當你的硬碟格式化為BTRFS後，此時你如果透過檔案總管打開時你只要還沒寫`/etc/fstab`它預設是不會掛載的，你點一下它就會掛載上來在`/media/$USER/$PARTITION`，這時你的所有操作都會直接在Default SubVolume，ID為5的根子卷做操作，如果你希望可以自動掛載或者使用BTRFS的特性，我強烈建議先做子卷規劃再做`fstab`設定。

1. 先做平鋪子卷規劃

   先把平鋪式的子卷建立好，如果你希望可以符合Timeshift要求或者希望這些子卷是易於辨識的，就把這些子卷使用`@`作為開頭，並挑選其中一個子卷作為主要用於存放資料的子卷，其餘同等級子卷在主要子卷上面先建立一個目錄，等編輯`fstab`再透過`subvol`將其掛載上去。

   如果你是像我一樣一開始把資料丟在Default Subvolume，只要把子卷建立好（並做好標記）然後把資料依序丟到子卷裡面就好，你可以先把掛載點先設好，然後如果有些資料會放在底下掛載其他子卷的目錄內，**不要**放進這個目錄，而是放到你預計要掛在這個目錄的子卷。
2. 設定`/etc/fstab`

   規劃完之後就可以去設定參數和掛載了，需要除了少數設定數值(比如`subvol`)大部分參數掛載參數只要是同一個磁區的就會跟著第一個設定的，所以如果你有甚麼需要的功能(比如你要開啟壓縮、`noatime`等)

   那在設定第一個子卷掛載時，第一個子卷應當是主要是主要的儲存子卷，剩下的跟著主要放資料的子卷掛載上去後跟著設定。如果第一個子卷掛載失敗，那你後續指定的其他子卷的掛載目錄會無法找到，進而整個分區的所有子卷都無法掛載。

   可以參考我放遊戲的Fury Renegade 2T設定完後的成果：
   ```bash
   UUID=12ae2bd7-fcc8-4cf7-9249-4ba4c823cac9 /disks/FURY_Renegade_2T       btrfs   subvol=/@FURY_2T,defaults,noatime,compress=zstd:3,discard=async       0 0
   UUID=12ae2bd7-fcc8-4cf7-9249-4ba4c823cac9 /disks/FURY_Renegade_2T/_Hgames       btrfs   subvol=/@Hgames,defaults,noatime,compress=zstd:3,discard=async        0 0
   UUID=12ae2bd7-fcc8-4cf7-9249-4ba4c823cac9 /disks/FURY_Renegade_2T/SteamLibrary  btrfs   subvol=/@SteamGames,defaults,noatime,compress=zstd:3,discard=async    0 0
   UUID=12ae2bd7-fcc8-4cf7-9249-4ba4c823cac9 /disks/FURY_Renegade_2T/_No_CoW       btrfs   subvol=/@No_CoW,defaults,noatime,compress=zstd:3,discard=async        0 0
   ```
   這個例子可以看到`@FURY_2T`是我主要存放資料的地方，然後我在`@FURY_2T`底下建立了`_Hgames`、`SteamLibrary`和`_No_Cow`用於掛載`@Hgames`、`@SteamGames`和`@No_CoW`。

   那成功透過`fstab`掛載後你使用subvolume檢查指令也可以看到對應子卷資訊：
   ```bash
   sudo btrfs subvolume show SteamLibrary/
   @SteamGames
        Name:                   @SteamGames
        UUID:                   69aef626-2884-e641-8e64-efc0e8b0815b
        Parent UUID:            -
        Received UUID:          -
        Creation time:          2025-12-11 18:54:11 +0800
        Subvolume ID:           259
        Generation:             24848
        Gen at creation:        23176
        Parent ID:              5
        Top level ID:           5
        Flags:                  -
        Send transid:           0
        Send time:              2025-12-11 18:54:11 +0800
        Receive transid:        0
        Receive time:           -
        Snapshot(s):
        Quota group:            n/a
   ```
   那如果是對沒有掛載的子卷的目錄使用該指令則會顯示`ERROR: Not a Btrfs subvolume: Invalid argument`

3. (可選項)在SubVolume底下建立SubVolume

   這麼做的目的更主要是在Take Snapshot時切割Snapshot區塊，因為每個SubVolume(無論等級高低)在BTRFS都視為不同的分區入口，所以如果建立巢狀SubVolume可以在Take snapshot會讓BTRFS視為「這裡有個SubVolume入口」從而切分需要與不需要快照的資料區。

4. (可選項)替不同子卷或目錄設定特定屬性

   那作到這裡，基本上你一個硬碟的初期規劃就差不多完成了，這裡說的設定特定屬性是指賦予目錄No CoW的屬性或者是特定目錄不須壓縮或者壓縮等級更高，這裡
   - 調整壓縮率：`sudo btrfs property set $PATH compression zstd:$LEVEL`
   - 關閉壓縮：`sudo btrfs property set $PATH compression no`
   - No CoW：`chattr +C $PATH`(關閉CoW) `lsattr $PARENT_PATH | grep $PATH`(檢查No CoW屬性，看到一堆`-`然後有個`C`就是) Ex: `---------------C------ ./_No_CoW`
   - 取得目錄設定：`sudo btrfs property get $PATH`

# BTRFS維護與優化指令
那如果你是像我這樣，一開始都是點檔案總管後掛載的，很大概率上這個分區其實是屬於未維護的狀態，空間越小和寫入量越大的情況下就越需維護

## BTRFS平衡
BTRFS在塞資料和Metadata的時候每次都會申請大量區塊(Chunk)，如果區塊被分配完了有可能發生`df -h`有空間但是寫入檔案會報錯`No space left on device (ENOSPC)`，這就代表Chunk被分配完了。那BTRFS有分為Data (你能使用的檔案、遊戲等)和Metadata(存放資料的Check Sum、目錄結構、檔案屬性)，如果你希望避免明明磁區還有空間但是卻無法存資料，你就會需要定時做Data Balance，指令如下：
```bash
sudo btrfs balance start -dusage=$NON_USAGE_PCT $PATH
```
`$PATH`指向你要做平衡的目錄（直接指向根會報錯）；`-dusage`指的是存放Data的Chunk佔用多少，任何低於這個參數的Chunk佔用都會被重新整理為一個新的Chunk，讓這些閒置空間過大Chunk可以重新被收回然後利用。

Metadata也可以被平衡，只是`dusage`變成`musage`，但一般來說Metadata佔用很小，並且這些資料可以說是BTRFS的命根子，所以可以的話定期使用`musage=0`或者不要太大的數字（比如小於10）用於回收沒有或者幾乎沒有被使用的Chunk就好，如果Metadata在平衡過程中壞掉輕則一些檔案**Checksum對不上**(檔案無法打開)，重則**B-Tree被破壞**，檔案無法訪問，所以非必要真的**不要**去平衡Metadata，尤其是`musage` > 50的情況，你不小心把你硬碟塞滿或者Chunk真的不夠再說。

那Balance跟接下來要說的defrage(重組)一樣，這個過程會對硬碟造成大量讀取甚至寫入，CPU也要計算B-Tree，所以在平衡的時候盡量不要一上來就用`dusage=60`這種參數，真的希望用到如此極致的平衡可以漸進式的做（比如30 -> 60、20 -> 40 ->60之類的），然後不需要全磁區平衡，除非你有使用新增設備擴充你的BTRFS Pool，這才會需要做全磁區Balance，這強制把資料分散在2個硬碟上。若有使用RAID和多設備會需要使用不同的參數，請自行翻閱[BTRFS docs - Balance](https://btrfs.readthedocs.io/en/latest/Balance.html)的說明。

> BTRFS全平衡是同時包含Data和Metadata平衡，正如前面所說，只有新增設備到BTRFS Pool的情況下才要全平衡。
>
> 如果BTRFS平衡到一半真的沒有Chunk用來搬運Metadata會造成The ENOSPC Loop這種檔案系統的Deadlock，這時候只能透過額外的Mount option `skip_balance`暫停失敗的平衡或插入USB加入BTRFS Pool，擴充臨時空間以完成平衡。

## BTRFS重組與壓縮
這個情況比較屬於BTRFS磁區你原本沒開壓縮，現在有開所以要做一次性優化。

像是我原本都用檔案總管掛載再使用，這次我設定完`/etc/fstab`有開啟壓縮，但是設定值只影響之後寫入的檔案，你如果希望你原本在硬碟上的檔案也可以被壓縮就要執行以下指令：
```bash
sudo btrfs filesystem defragment -r -v -czstd $PATH
```
這個指令會強制重組(`defragment`)這個磁區的檔案，`-czstd`代表強制用zstd壓縮檔案，真的不能壓就存回原檔，類似`compress-force=zstd:3`。

做完壓縮後，你直接`df -h`可能無法直觀的看到壓縮成果或者具體細節，你可以用`compsize`來查看，這屬於非預裝工具，我留到[BTRFS其他套件與空間優化](./BTRFS其他套件與空間優化.md)在解釋

> 需要注意的是重組會強制把資料變成連續資料段，SSD對資料碎片化不敏感以外，重組會破壞reflink，導致你的硬碟空間利用大幅上升（在你使用越多reflink複製檔案或者你有去重時更為明顯），而且重組本身也會對 CPU (尤其又有使用壓縮)和硬碟造成大量IO，這也會提昇SSD的磨損

# 下一步
那做設定和維護後，就可以處理快照和去重(Deduplication)了，在去重之前我建議先學會和設定[快照](../Snapshot/快照使用和TimeShift設定.md)

# Reference
- [Arch wiki - BTRFS](https://wiki.archlinux.org/title/Btrfs)
- [Arch中文wiki - BTRFS](https://wiki.archlinuxcn.org/zh-tw/Btrfs)
- [BTRFS documentation](https://btrfs.readthedocs.io/en/latest/)
  - [Subvolumes](https://btrfs.readthedocs.io/en/latest/Subvolumes.html)
  - [Balance](https://btrfs.readthedocs.io/en/latest/Balance.html)
- [Oracle Linux Blog - Exploring the Discard Mechanism of Btrfs Filesystem](https://blogs.oracle.com/linux/btrfs-discard)
- `man 5 btrfs`
- [Btrfs balance stuck and no way to resume or cancel (No space left)](https://www.reddit.com/r/btrfs/comments/zfdbvz/btrfs_balance_stuck_and_no_way_to_resume_or/?show=original)