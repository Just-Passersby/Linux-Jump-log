# 建立Blog紀錄
waaaaaaa！！`.net`買起來不便宜欸！！算上稅金買完快400！`.com`還便宜3、40元，但是`.com`太沒個性了！我還是覺得`.net`比較適合我，雖然最近研究OS比較多，但我可是研究Networking起家的

## 前端選擇
我先說，這些技術純粹是為了Blog才學的，我並不想跟別人卷前端(我也卷不贏)，所以想走前端的不要看我的前端技術棧。

有了這個前提，我們可以拋棄掉React了，這東西太肥，我不喜歡，而且React的能力拿來做Blog屬於殺雞焉用牛刀。

你問我有甚麼比較喜歡的前端技術？我是三件套派的，基於原生標準擴充的技術就是HTMX + Svelte + Lit，這些技術其實也很不錯，我先說說三件套的問題吧。

三件套寫Blog不是不行，但我的文章都是Markdown文件，變成我必須花時間改成HTML，以及日後網頁怎麼編譯也要自己處理，維護起來花時間

那HTMX + Svelte + Lit呢？恩，Blog沒有後端的問題，HTMX優勢發揮不出來；直接用Svelte等於自組框架；Lit是Web Component，我除了Blog要主題以外基本沒有要重複設計的元件，都是大材小用。

那果然還是框架方便吧？但是React不是被我拋棄了嗎？Next.js也用不了了，選啥？

嘿，你還別說，Blog前端生態跟「專業」前端是2個生態，除了前霸主WordPress以外，Hugo、Ghost都是常見或優秀的選擇，但目前更流行且近乎最佳解的，是**Astro**這個框架。

### What and Why Astro
作為一個家裡空間不大，沒地方擺電腦做Reverse Proxy + Web Server，同時還要擔心電錶轉爆，這意味著我不能Self host (至少目前不能，而且真的Self host安全性設定我有的受)，此時Ghost出局，因為必須Self host。

那Hugo呢？我還有朋友用Hugo欸！Hugo確實不錯，基於Golang，編譯性能好到飛起，但就是Golang我沒學過，Template寫起來比較麻煩，而且我有計畫弄一些比較不一樣的Blog分類方式，在Hugo上會比較難做，有些功能一旦Hugo沒有擴充，我將要花上大量時間來處理，更不要說還有三件套在上面。

至於Astro，針對Blog場景做優化，載入速度極快，基本沒有JS，真的需要JS或互動功能，可以透過群島式架構局部載入JS組件(比如React、Svelte)，目前的生態也相對豐富，在Style上，Astro你不套Template也有簡單的可以自己來設計，不用完全從CSS手刻，甚至想要自己實現額外擴充也很容易，最重要的，**內建**Markdown解析器，會寫JS的連Markdown + JSX (`.mdx`)相容性都非常高，要說限制吧，超過數千篇Blog編譯比較久，假設我日更Blog，大概要2.5年才能超過1k篇，從我目前對我筆記和Blog的Commit紀錄來看，我1個月也才1、2篇，恩，很夠了。

欸，你剛剛說你不要Self host Blog，那你要架到哪裡？那當然是雲託管阿，目前Blog的雲託管大家最熟知的肯定是GitHub pages (`username.github.io`)，但GitHub Pages要求你的Blog (準確來說是`username.github.io`這個Repo)必須Open Source (至少Free Tier是這樣)。幹麼？Blog不能Close Source嗎？還有就是他只會顯示main的網頁，你如果分支開測試只能另外部署Server、環境和編譯才能測試。

既然不用GitHub Pages，那另一個選項是誰？那就是性能更好的Cloudflare Pages，指定好GitHub Repo (無論Public還是Private)就會自動編譯，編譯好之前別人會看到舊網頁，你甚至可以開分支來測試網頁效果，Cloudflare Pages會自動建立一個URL給你測試，別人可以正常連你的Blog，等你開發和測試完了再Merge，在Astro的官方建議中，就是最建議搭配Cloudflare Pages使用。

Vercel也是非常強力的候選，畢竟業界流行Next.js很大原因就是Vercel + Next.js絲滑搭配，但缺點也很明顯，Free Tier有流量限制，太多人請求會有429錯誤，Cloudflare Pages不搞這套，挺爽的。

## 環境部署
技術選型完成後，就要來佈置開發環境，Astro需要Node.js的開發環境，既然在Python有PyProject和Astral uv，那Node.js也有，目前的主流是Fast Node Manager，主要用Rust開發，速度很快。

> fnm安裝腳本依賴：`curl`和`unzip`

Linux安裝和更新：
```bash
# Install
curl -fsSL https://fnm.vercel.app/install | bash

# Upgrade
curl -fsSL https://fnm.vercel.app/install | bash -s -- --skip-shell
```

macOS也差不多，但是macOS也可以用以下方式安裝和更新：
```zsh
# Install
brew install fnm

# upgrade
brew upgrade fnm
```
fnm安裝完後就是設定fnm的變數了，根據Shell來決定初始化fnm的變數

bash:
```bash
echo 'eval "$(fnm env --use-on-cd --shell bash)"' >> ~/.bashrc
```

zsh:
```zsh
echo 'eval "$(fnm env --use-on-cd --shell zsh)"' >> ~/.zshrc
```

fish:
```bash
echo 'fnm env --use-on-cd --shell fish | source' >> ~/.config/fish/conf.d/fnm.fish
```

接下來重開終端機或者輸入`source ~/.bashrc` (看Shell)後fnm就設定完成，接下來安裝Node.js：
```bash
fnm install --lts          # 裝最新 LTS Node.js
fnm use --lts
node -v                    # 確認
```

為了降低硬碟使用量，後續使用`pnpm`管理依賴，使用以下指令安裝`pnpm`：
```bash
corepack enable
corepack prepare pnpm@latest --activate
```

## Blog初始化
終於設定完環境，來初始化Astro專案吧，使用VS Code開發的話，可以先新增[Astro擴充](https://marketplace.visualstudio.com/items?itemName=astro-build.astro-vscode)

雖然Astro Docs是說可以建立在任何地方，如果不在空目錄建立可以自動幫忙建立目錄，但我還是保有建立專案時先建立空目錄，總之輸入以下指令開始初始化專案：
```bash
pnpm create astro@latest
```
接下來會有一些選項，我挑比較重要的講
- Template: 
  ```bash
  How would you like to start your new project?
         ○ A basic, helpful starter project     # Astro的Hello world，專門學Astro用的
         ● Use blog template                    # 我要創Blog，選這個
         ○ Use docs (Starlight) template 
         ○ Use minimal (empty) template 
  ```

流程參考：
```bash
 astro   Launch sequence initiated.

   dir   Where should we create your new project?
         ./extinct-escape

  tmpl   How would you like to start your new project?
         Use blog template

  deps   Install dependencies?
         Yes

   git   Initialize a new git repository?
         Yes

      ✔  Project initialized!
         ■ Template copied
         ■ Dependencies installed
         ■ Git initialized

  next   Liftoff confirmed. Explore your project!

         Enter your project directory using cd ./extinct-escape 
         Run pnpm dev to start the dev server. CTRL+C to stop.
         Add frameworks like react or tailwind using astro add.

         Stuck? Join us at https://astro.build/chat

╭─────╮  Houston:
│ ◠ ◡ ◠  Good luck out there, astronaut! 🚀
╰─────╯
```
這裡提供了關鍵資訊，執行`pnpm dev`可以測試網頁，不過他是真的會在當前目錄再建立一個專案資料夾(我用預設的`.extinct-escape`)，所以用預設目錄的記得`cd`進去，跑起來後預設會是`localhost:4321`，看到網頁就是成功。

網頁成功後就可以開始弄GitHub Repo，如果前面`Initialize a new git repository?`是Yes他會幫你生好`.gitignore`(針對Astro的)和First commit，所以建立一個空的Repo就好，不然要處理merge問題，弄好就可以設定Cloudflare Pages了。

## 建立Blog網頁
打開Cloudflare Dashboard，首頁中間有個**Workers and Pages**，按下Start build，選擇Connect GitHub，你如果懶惰不是不能授權整個帳號，但保持最小原則概念，我這裡只授權我的Blog儲存庫。

接下來因為Pages和Worker已經被整合了，如果你問AI大概率是舊資料(至少3/11凌晨問Sonnet 4.6和Gemini 3.1 Pro是這樣)，他現在會非常智能的抓環境，自動處理好build指令，Deploy指令預設也會生成好，不須修改，build的時候會自動安裝依賴，build完成後會自動跳轉，我自己就沒碰到錯誤，Build完後會發PR，用於讓你的Blog日後可以順利的在Cloudflare worker部屬，如果跟我一樣是靜態網頁，那Merge完後要修改`wrangler.jsonc`
```json
{
  "name": "PROJECT-NAME",
  "compatibility_date": "2025-09-27",
  "assets": {
    "directory": "./dist"
  }
}
```

此時就可以來綁定Domain了，在Worker右邊有個Domain & Routes，點進去後在第一行的Domain & Routes的右邊有個`+Add`，點進去綁選Custom Domain，綁定給人看Blog的網域，如果DNS是從Cloudflare買的或者代管，Cloudflare會自動幫忙處理好Worker和Domain的關係。

## Blog規劃
這個規劃是我自己有在瀏覽Blog時的想法，我對Blog Style風格很大程度受到[Ivon Huang](https://ivonblog.com/)啟發，所以會想做的偏簡潔一點，首頁就是Blog介紹、Banner然後下面就是文章。

但可能我逛的Blog也不多，總之我目前逛過的Blog有下問題：
- 標籤太多，一旦經營時間增加標籤很難找，英文這種情況可以依照字母排序，但是中文不行
- 文章難找，有的Blog找文章甚至是直接導向到Google使用`site:`參數

總結來說痛點都是圍繞在**找過往文章**這件事上面，那我就順便把我除了解決痛點的想法，也把我想怎麼設計Blog的想法列在底下：
- 網頁顏色切換：淺色會以水藍色作為主色調，深色會以夜空藍作為主色調
- Blog預設用時間軸做排序，但也會有系列文，預計以樹狀呈現
- Tags會用分組的方式處理，盡量縮減找標籤的痛苦
- 閱覽文章時，左側放置TOC跟隨網頁滑動，右側留空
- Blog導航留在最上面，跟隨滑動再考慮

那可能會做但是不會馬上做的：
- 留言板
- 更多特效或動畫
- 筆記獨立一欄，跟Blog共用檢索系統(標籤)，但是具備獨立性，避免搜尋Blog時搜到筆記

## Astro編輯時要注意的問題
雖然支持Markdown，但是開頭要有以下內容：
```markdown
---
title: 'Markdown Style Guide'
description: 'Here is a sample of some basic ...'
pubDate: 'Jun 19 2024'
heroImage: '../../assets/blog-placeholder-1.jpg'
---
...內文...
```

然後`[toc]`雖然在HackMD和VS Code Markdown Enhanced Preview會自動解析，但是Astro有自己的另一套解析機制，所以必須拔除。

大概就是這樣吧，有缺再回來補。

# Reference
[fnm GitHub](https://github.com/Schniz/fnm)
[Astro Docs - Get started](https://docs.astro.build/en/getting-started/)
[Cloudflare docs - Pages limits](https://developers.cloudflare.com/pages/platform/limits/)
[Vercel Docs - Limits](https://vercel.com/docs/limits)
