---
title: shell 文件操作
date: 2021-07-07 17:20:26
tags: [linux,file]
---



## 读行

```
while read line ;do
#or while read -r line ;do
echo $line
done < $1
```

```
cat $1 | while read line ;do
echo $line
done
```

使用for时，结果略有不同, for以空格为一行
```
for line in $(cat $1) ;do
echo $line
done
```

## reference

https://bash.cyberciti.biz/guide/Reads_from_the_file_descriptor_(fd)