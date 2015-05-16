---
layout: postlayout
title: 自动生成26个英文字母的序列
categories: [excel]
tags: [excel,char]
---
自动生成26个英文字母的序列

=SUBSTITUTE(ADDRESS(1,MOD(ROW(),270),4),1,"")

![](http://files.c.excelhome.net/forum/201109/23/1126258xd08uxg9dlc9d00.gif)

![](http://files.c.excelhome.net/forum/201109/23/113652w8dwxne7fgo7xz2o.gif)