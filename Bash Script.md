# 迴圈
```bash
for i in {1..5}; do
	echo "Number: $i";
done
```

# 環境變數
```bash
export VAR="something"
```

若讀取的檔案名稱內有空格，會自動把他們拆開來變成不同的檔案。所以要記得加上 `""`。

浮點數的小工具：`bc`

```bash
#!/usr/bin/bash

total_len=0

for file in *.mp4 ; do
	duration=$(ffprobe -loglevel error -show_entries format=duration -output_format default=noprint_wrappers=1:nokey=1 "$file")
	total_len=$(echo "scale = 10; $duration + $total_len" | bc)
done

total_len=$(echo "scale = 10; $total_len / 2" | bc)
echo $total_len
```

1. `for file in *.mp4` 直接變成 `find`，好爽喔
2. `ffprobe` 可以用來處理影片的元資訊
3. 檔案名稱一定要加上 `""`
4. 浮點數運算不能直接加，最好使用像是 `bc` 的小工具來處理