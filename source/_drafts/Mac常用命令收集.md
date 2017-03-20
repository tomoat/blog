---
title: Mac常用命令收集
---

## Mac OS X 下修改文件属性：创建时间、修改时间

- [[YY]YY] 四位数年
- MM 两位数月（01~12）
- DD 两位数日（01~31）
- hh 两位数时（00~23）
- mm 两位数分（00~59）
- [.SS] 两位数秒（00~59）

------

## 修改文件的“创建时间”属性

打开Terminal，首先输入以下内容，暂时不要回车：

```
touch -t [[CC]YY]MMDDhhmm[.SS]
```

将YYYYMMDDhhmm 要替换成期望的时间，比如 201209160330.00 //即为2012年9月16日凌晨3点30分0秒。 其中[CC]YY可以缺省前两位“世纪”*，只输入后两位年份，并且秒钟[.SS]也可缺省*。把需要修改的文件拖到 Terminal 窗口，这时文件的路径会自动补全，当然你也可以自己手动输入文件路径。
ex：

```
touch -t 201209160330.00 /Users/name/Desktop/somefile.jpg
```

回车即完成修改。

------

## 修改文件的“修改时间”

步骤和上面修改“创建时间”基本相同，只是把参数 -t 改成 -mt
// time 和 modify time
ex：

```
touch -mt 201209160330.00 /Users/name/Desktop/somefile.jpg
```

还有批量修改文件日期的方法可以看一下[这里](http://danilo.ariadoss.com/howto-change-date-modified-date-created-mac/)。
*世纪实际是世纪-1，20xx年是21世纪。
*缺省年份则自动为当年。
*缺省秒则自动为00秒。
*当修改时间早于创建时间，Mac OS X显示创建修改均为最早时间，Windows会分别显示不合逻辑的时间，这样会露馅，记得修改时间要晚于创建时间。