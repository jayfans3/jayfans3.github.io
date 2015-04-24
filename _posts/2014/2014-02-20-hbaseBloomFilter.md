---
layout: post
title: hbase使用bloomFilter
categories:
- 人
tags:
- 书
---



#hbase使用bloomFilter

get：

- 无论是row，rowcol，都能起作用，但是要get要匹配到哪一个bloomFilter.type上
- row是一条条的在hfile中过滤，真正的过滤


scan:

- 不指定cf.qulifer不起作用，因为不能确定范围。row+col+qualify
