# WinBTRFS Mapping
這是個血淋淋的教訓...我有一天只是跟朋友打完Trakov後在運行Windows的情況下打開女神任務玩一下，結果隔日回到Ubuntu來玩發現我的存檔存不了，跟我說無法打開指定路徑，然後想說有自動儲存，結果發現自動儲存也失效，最後點進遊戲的存檔資夾發現我要存的位置和自動儲存的`owner:group`變成了`nobody:users`，God damn，關卡重打，裝備重刷......，總之這場血淋淋的教訓讓我深究原因。

一番調查後，發現winBTRFS有Mapping機制，如果你是直接在Linux掛載NTFS你會發現你可以直接用，甚至Windows權限怎麼設定不影響你的檔案權限，原因是Linux並不認識UAC，所以ntfs-3g在讀取檔案時會直接看誰先掛載這個磁區，誰先讀去來決定權限開給誰。但是BTRFS是Linux原生的檔案系統，而不事項NTFS這類外來種，所以winBTRFS會使用Mapping的機制將Windows的使用者與Linux使用者和群組進行對應，若在不設定的情況下任何User會對應`nobody`，Group會對應`users`，此時需要去設定winBTRFS的Mapping才可以避免這個問題。

在對應之前你需要先取得3個數值：你要Mapping的Linux uid、gid和Windows的SID，這樣才能完成`owner:group`的映射。

Windows的SID可以粗略的理解為Windows Users的uid，取得SID的方法是在PowerShell輸入`whoami /user`(GitHub頁面的指令被Windows以遺棄，無法使用)，然後Linux取得uid和gid只要輸入`id`就可以了。

3個數值都取得後就要在Windows打開註冊表，然後找到`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\btrfs`底下會有`Mappings`和`GroupMappings`，分別在裡面建立一個DWORD，名稱填入你的SID，Value的部份統一皆是十進制，並在`Mappings`和`GroupMappings`分別填入你的`uid`和`gid`，完成後把硬碟卸載後重新掛上或者直接重新開機，之後你的寫入至BTRFS詞逡的檔案就會被正確Mapping，不會再是nobody和users了。

但要注意的是，如果是Mapping就寫入的檔案，你只有把檔案刪除重寫或者在Linux提權後設定`owner:group`才能讓你正常訪問檔案，下次如果有安裝winBTRFS記得要做好Mapping了，出此使用的或許也可以參考這篇筆記。

# Reference
[Ivon - Windows系統如何掛載Linux的BTRFS硬碟：使用WinBTRFS](https://ivonblog.com/posts/winbtrfs-usage/)
[GitHub - winBTRFS](https://github.com/maharmstone/btrfs)