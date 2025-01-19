# 使用snap管理套件
Snap作為Ubuntu內建的套件管理程式，背後也有Canonical推動，因此可以作為Ubuntu管理套件包的一種方法

[toc]

# 關於Snap的爭議
我個人認為跳轉為Ubuntu，Snap的爭議是值得關注的，但不可否認Snap也是現階段非Arch，尤其Ubuntu體系中很有競爭力/方便的套件管理工具，畢竟有Canonical撐腰（至少現在撐的住）

但畢竟這裡是Linux，開源、選擇很多的地方，所以Snap說到底也只是管理套件的一種手段，你想的話也不是不能在Debian上用dnf管理套件

扯遠了，Snap爭議與討論我認為可以參考[Ivon的文章](https://ivonblog.com/posts/linux-snap-pros-and-cons/)，個人認為講的還蠻詳細的

# Snap的安裝與使用
如果是沒有內建Snap的發行版，可以參考[這篇文章](https://ivonblog.com/posts/linux-snap-introduction/#2-%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%9Dsnap%E8%BB%9F%E9%AB%94)了解如何安裝，以下內容也是參考這篇文章的，詳細用法用解釋也考以參考該文章

- 列出透過已snap安裝的軟體：
 ```bash
 sudo snap list
 ```

- 搜索特定套件（以Discord為例）
 ```bash
 snap search discord
 ```

- 透過snap安裝套件，以Discord為例：
 ```bash
 sudo snap install discord
 ```

- 更新透過snap安裝的軟體：
 ```bash
 sudo snap refresh
 ```
 如果是要開啟或關閉自動更新功能，透過添加`--unhold`和`--hold`來控制，如果要控制個別套件是否啟用自動更新只須再添加套件名稱就可以
 ```bash
 # 禁止所有套件自動更新
 sudo snap refresh --hold 

 # 允許所有套件自動更新
 sudo snap refresh --unhold 

 # 禁止特定套件自動更新（以Discord為例）
 sudo snap refresh --hold discord
 ```

- snap移除軟體（以Discord為例）
 ```bash
 # 僅刪除套件
 sudo snap remove discord

 # 連同使用者資料一同刪除
 sudo snap remove --purge discord
 ```

Snap還能控制安裝的軟體的權限、存放位置等功能，可以參考[Ivon的文章](https://ivonblog.com/posts/linux-snap-introduction/)，我有用到再回來寫或重新整理XD

# 部份軟體用Snap裝遇到的問題
微軟雖然有在Snapcraft上架VS Code，但是有些套件需要提權的操作（比如background）會無法正常使用，稍微查了一下目前暫時無解，所以我還是使用.deb的方式安裝VS Code

# Reference
[Ivon - Linux系統的Snap是什麼？跨發行版軟體套件管理員使用方法](https://ivonblog.com/posts/linux-snap-introduction/)
[Ivon - Ubuntu Linux用Snap安裝軟體的優缺點](https://ivonblog.com/posts/linux-snap-pros-and-cons/)
[Reddit - Flatpak or Snap?](https://www.reddit.com/r/linuxmasterrace/comments/u4te9z/flatpak_or_snap/)