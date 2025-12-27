# BTRFS其他套件與空間優化

觀察壓縮率套件：`compsize`或`btrfs-compsize`
全目錄/硬碟壓縮：`sudo btrfs filesystem defragment -r -v -czstd $PATH`
壓縮成果（壓縮前是100%）：
```bash
Processed 181696 files, 1231950 regular extents (1238287 refs), 22047 inline.
Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       93%      625G         667G         669G       
none       100%      577G         577G         579G       
zstd        53%       47G          89G          89G       
prealloc   100%       16M          16M          16M    
```

去重工具：`sudo apt install duperemove`
