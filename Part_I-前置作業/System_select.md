# Linux發行版選擇
自高中以來由於接觸的Linux發行版已Debian (和其他分支)用得最多，所以心中Linux發行版首選基本上是以Debian體系為主，不過在開始虛擬機體驗並測試後，雖然心中首選主要還是以Debian體系為主，但多出了幾個其他特別的Linux發行版

[toc]

# Linux發行版心中排行Top 5
1. Ubuntu
2. Ubuntu Budgie
3. Mint Linux Ubuntu based
4. Debian
5. Solus Budgie

其實前2名都是一樣的系統，不同的是使用的桌面環境，Budgie在Ubuntu上看起來十分漂亮，看起來可自訂性更高(不算額外安裝的套件的話)

然而Ubuntu budgie在VMware workstation Pro 17.6.0上運行表現十分糟糕，桌面環境容易崩潰，即使RAM給到了6G也是光運行基礎系統就是吃滿狀態，在測試Steam遊玩時沒有聲音輸出

但在實際爬文上Budgie理論上不會和GNOME 3有著性能上的差異，甚至Budgie在某些情況下比GNOME 3更省資源，所以我猜測是虛擬機的問題

Mint Linux和Debian則是也有各自的優勢，一個是類似Windows的操作而且同樣有巨大的社群和Ubuntu的套件池支撐，另一個則是一整個大體系的起源，同時也是最熟悉的作業系統，不過前者可以有著類似Ubuntu的一用程度，後者則是最乾淨，但也是需要花最多心思調教的作業系統

Solus Budgie很有意思，是在體驗了Ubuntu Budgie去搜尋了這個桌面環境後發現的作業系統，Budgie為了Solus這個系統而生

一開始也是基於Debian (名叫Solus OS)，後來獨立更名為Evolve OS後改用eopkg作為套件管理工具，並在更名為Solus後Budgie這個桌面環境也從不穩定的更新變成迅速跌代，能與GNOME持平的桌面環境

而Solus搭配Budgie也是十分流暢，同時跟在Ubuntu上運行一樣，都是十分漂亮，不過個人感覺預設風格更加貼近Mint Linux這類設計，而其包管理工具eopkg在Reddit的評論是跟apt十分類似，學習、跳轉成本低

# 其他特別的選擇
雖然說這些發行版不是心中首選，但會是個若未來有時間和多的機器，或者是上述的發行版不想用了之後會想跳轉的版本

- Arch Linux
Arch Linux一來是我有認識的人使用，二來SteamOS是基於該發行版修改而來，因此若有玩遊戲或者其他問題，這個發行版也相對更容易找到解決方法

- Bliss OS
這個發行版更加特別，是唯一還有在維護的x86 Andriod，同時也有對電腦的使用環境做優化，而且可以選擇有Google框架的版本，因此這個系統即便沒有使用在實體機上，未來打算使用KVM運行該系統來作為Linux上面的手機模擬器(不過Bliss OS官方文檔上明確註明虛擬化系統效率不高，在VMware上確實碰到安裝完無法進系統的問題)

- FreeBSD
雖然也被歸類在Unix-like，但我個人認為在發展史上撇除法律因素，這可以說是我個人認為最正統的Unix系統了，由於當初與AT&T的法律糾紛錯過了發展的大好機會，因此支援度廣泛度甚至不及Linux，但作為(個人認為)最正統的Unix還是想要把玩一下

