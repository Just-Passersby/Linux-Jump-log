# Ubuntu用了1年又1月的心得

其實大概上個月就是我Linux日常使用正式1年了，只是上個月不是大學期末考、實驗室計畫不然就是在愁BTRFS Snapshot的筆記怎麼寫和實驗會更好，到了快1月底又有明日方舟：終末地，嘿嘿不好意思拉電線拉到上癮，結果就這樣拖到現在，我決定趁我有動力先把一些心得寫一寫。

## 自身Linux使用體感
稍微觀察我的Git紀錄以及內文應該可以注意到我從Ubuntu-Budgie跳轉到Kubuntu對吧？這有甚麼區別嗎？有，區別可大了！

我先說說Ubuntu-Budgie為何被我替代：
1. 遊戲Crash帶走整個系統
  
  主要是我玩遊戲，尤其是War Thunder、CS2(雖然這款現在沒玩了)這種佔據整個螢幕且很吃效能或畫面複雜的遊戲一旦崩潰不只遊戲本身崩潰，還會連帶把我的**DE搞崩**，DE崩了**系統跟著崩**...(至於DJMAX雖然也是全螢幕但是不吃效能所以不受影響)
  
  後來我朋友分享他在Fedora玩CS2也會Crash，但是只Crash遊戲，我後來問他DE是GNOME on Wayland，因此我把問題定位到了X11身上。

2. 安裝遠端桌面和更新把自己弄爆
  
  這其實算是Ubuntu-Budgie本身也有問題，總之我碰到了這些問題
  - 安裝Chrome Remote Desktop把檔案總管和硬碟掛載搞爛
     
    安裝完成Chrome Remote Desktop後檔案總管這邊無法直接掛載非系統磁區，如果寫`/etc/fstab`會直接從檔案總管左邊的Device消失，只能自己去找路徑...

    更要命的是非系統套件，比如Steam會無法讀取非系統碟磁區，這導致我很多遊戲無法讀取。

  - Kernel更新容易出問題
    在Ubuntu-Budgie上很容易遇到`VFS: Unable to mount root fs on unknown-block(0,0)`這個錯誤(可以參考[Ubuntu半年使用心得](./Ubuntu半年使用心得.md))，雖然問題很好解決，但是更新5次碰到2、3次就算有經驗還是很令人糟心，我受此影響跑去學習LFS看有沒有解方(但很顯然那裡只要會Ctrl + C和Ctrl + V，且涉及GUI要到BLFS，所以沒有)。

不過`mount root fs`的錯誤因為我使用Kubuntu只有經歷過一次HWE更新，雖然沒有錯誤我也很難說Kubuntu是不是真的很穩，反正我[這篇](./Ubuntu半年使用心得.md)有提到解決方法，所以為了Wayland (KDE Plasma 5可以用Wayland)以及使用BTRFS來保障系統穩定性我決定重灌Kubuntu (當然Ubuntu的BTRFS設定跟蠻荒一樣，要自己去設定，至少Subvolumes預設切的很適合TimeShift)，至於我當初跳到Kubuntu的其他心路歷程以及我當時碰過得坑可以參考[這篇](./關於切換到Kubuntu那些事.md)，裡面還有談到EFI分區搬移的問題(該死的Windows安裝時強姦Linux設定的EFI分區)。

另外在這一年我還不小心當起了Linux傳教士，但凡是個資工相關的同學或朋友，碰到這陣子SSD太貴或是Windows又出問題的不然就是電腦性能怪怪的我都會跟對方說「你該用Linux了」XD，我身邊還真的有2個人因為我的建議之下考慮雙系統或主力使用Linux (當然也跟25H2的更新過於垃圾有關)，期望Valve多發力，把Proton做的更好，不然我玩癡情哥哥與病弱妹妹的鄉間生活不能好好玩QQ，看來我該研究GE-Proton如何安裝到Steam了 (我平常都是用Bottles玩遊戲比較多)。

## Linux適合日用嗎？
那這一年使用下來如果問我Linux適不適合日用，我的答案其實又跟半年前不太一樣，那就是「適合」，但具體情況看發行版，Ubuntu為例，Ubuntu-flavors因為除了修改DE以及底下開發團隊不屬於Canonical，更多是合作，這就導致能列入Ubuntu-flavors雖然理論上都是Canonical的APT Source，但實際預設安裝的內容以及部份組件的更新一定會與Ubuntu原版不同，Ubuntu-Budgie這看起來新穎且頗具潛力的發行版我認為就是個例子，更新Kernel容易失敗且Budgie本身主要建立在Solus又因為是很新的DE開發團隊所以能維護的人非常有限。

- 適合新手入坑的發行版：Ubuntu與Ubuntu-based發行版Zorin OS (Windows難民營)與Mint Linux (前Windows難民營與現任Snap反對基地)
  
  隨著ZorinOS 18的推出，他的界面風格以及預裝Wine讓使用者可以無縫跑`.exe`的設計成為最新Windows 10停止支援後的難民聚集地，經過時間沈澱後也不少ZorinOS使用者正式融入Linux社群成為日常使用者。
  
  至於Mint Linux則是Windows XP時期的難民聚集地，但是隨著Canonical強推Snap以及不開放或開源Snap Store後端使得大量Ubuntu使用者生氣又一波人跑去Mint Linux，因此現在Mint Linux更主要成為Anti-Snap的Ubuntu-based使用者聚集地，也因為Mint Linux口碑足夠好，Mint Linux也一直是發行版使用人數排行長期居住在前三名的發行版。

- 進階使用者，不畏懼動些手：Fedora、Debian
  如果有一定動手能力那Fedora和Debian也是個強而有利的選擇，前者是RHEL上游你想體驗很新但又不會像Arch一樣很容易把自己更新更爆值得考慮，但是我個人不太喜歡這種半年一大更。
  
  後者則是宣佈允許非開源軟體庫加入到Debian後終於Debian使用者可以不用擔心日用很難了，硬體也不用挑三揀四挑能吃開源驅動的硬體，只要你用Stable那就是一個非常乾淨且穩如老狗的發行版(真的老，我記得Ubuntu的套件庫是建立在Debian test之上，但不要因為Debian還有個開發上游Sid就不代表Test沒有非常新鮮剛編譯好的套件)。

- 極致掌控、不新不開心或者想當自我挑戰、被當大神：Arch、EndeavourOS、CachyOS
  如果你很有自信真的想用Arch系列自我挑戰或者你就是覺得套件不新你不用不然就是你就是想說「I use Arch btw」那除了Arch以外，你想簡單一點EndeavourOS也是個熱門人選，如果你想真正成為倍受追捧(或迷因內)的存在，好好用Arch練練功吧，只要肯看Wiki就能拼的出來，EndeavourOS只是簡化這一過程。
  
  或者你有Zen 2以及之後的處理器(同代或更新的Intel也可)後起之秀CachyOS也是非常優秀的選擇，使用Linux最新效率最高的BORE排程器且主要使用x86-64-v3/v4或Zen4指令集來編譯軟體提昇使用效率，然後也幫你簡化Arch系列Linux安裝(或者你就是純粹Arch精神可以用CLI安裝)，最主要，內建BTRFS + Snapper + BTRFS Assistant，更爆直接回滾！至於你說這發行版太新怕團隊倒？贊助商直接掛著Framework、CDN 77和Cloudflare，看起來不太需要擔心！

- 被低估、冷門但是穩定很有實力的存在：openSUSE
  既然剛剛都提到了BTRFS，如果你不畏懼動手但你討厭Key指令，openSUSE也是非常值得考慮的選項，YaST提供類似Control Panel的界面讓你點，不過據說YaST會跟文字檔有衝突，這算是小麻煩。
  
  我的首選與手推肯定是OpenSUSE Tumbleweed，因為他是滾動更新，所以軟體庫也是很靠近上游的發行版，但是SUSE替openSUSE Tumbleweed有做一個OpenQA的測試，這使得openSUSE Tumbleweed是個雖然版本號不固定跟Arch一樣但是經歷過高強度測試穩定才會推下來，再來BTRFS也是SUSE深耕的檔案系統所以BTRFS使用體驗上以及如何設定與修復等SOP在SUSE系列的發行版絕對是最頂尖的存在，[官方wiki](https://en.opensuse.org/SDB:BTRFS#How_to_repair_a_broken/unmountable_btrfs_filesystem)甚至還有辦法教你處理BTRFS不能掛載可以怎麼修。
  
  不過SUSE系列的發行版最尷尬的是因為討論度很低所以他能添加的庫和第三方軟體不多，除了要借用Red Hat的rpm以外使用上也更需要DistroBox這類工具彌補軟體庫太少的問題。

你問我接下來會換發行版嗎？其實我有在考慮，我會去CachyOS
- DE被限制
  
  這是主要的，我安裝Kubuntu的時候我感受到甚麼就做LTS的「限制」，明明KDE Plasma 6已經很穩定且Wayland表現不錯，但是因為Ubuntu 24.04 LTS凍結軟體庫的時候KDE Plasma 6剛出，所以KDE只能推出5.27版，導致我有9070XT同時又深受X11之苦卻只能用這個對Wayland還有實驗性的DE

- 對快速迭代的擔憂
  
  如果跑去用Ubuntu non-LTS那我還不如用Fedora，我不喜歡這種不到1年就要大版本更新的原因主要是對我而言大更新伴隨套件會有大改革，這會導致維護成本的提昇，Ubuntu LTS或Debain不是沒這問題，還會隨著時間跨度放大問題，但他們除了時間跨度夠大以外，還會持續替這些老版本出安全補丁但Ubuntu non-LTS和Fedora大版本一出來會直接被拋棄。

- BTRFS + Snapper兜底與性能優化
  
  最後因為學會BTRFS以及有做LFS經驗搭配我有9900X這種Zen 5新平台最後一直被推薦去使用CachyOS，如果CachyOS沒有性能上的優化和BTRFS + Snapper兜底的話，我個人離開Debian家族後我會跑去用變色龍家族OpenSUSE Tumbleweed體驗又新又穩的系統。

## 想回Windows嗎？
老實說，我因為電腦CPU其實夠強，加上我現在使用Windows的時間越來越少，截止至現在打文章的時候我已經至少2個月沒開Windows了，每次開Windows不是因為我玩特戰英豪或Apex (這兩款我根本沒玩)，而是為了跟朋友用Radmin VPN玩遊戲，我接下來TimeShift設定完成後下一步應該會是把我剩下2顆NTFS硬碟也格式化成BTRFS，順便讓被Windows佔用的SSD釋放出來讓這1T可以自由運用，同時運用BTRFS的CoW特性讓我的SSD可以作到1T存個1.25T，或者1T可以存個1.1T在這SSD暴漲的時候都是勝利，這也是我推薦我朋友Linux的第二個原因XD至於需要Windows甚至跑遊戲怎麼辦？那就用Virtual Machine就可以解決了，要跑遊戲就設定VFIO，真的炸掉刪掉重灌就好，只要Linux不炸(也比Windows難炸)就好。
