---
title: Day_05 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 数据的排序

> reducer的排序为单节点排序 
> 使用新的分区策略解决多个reduce的排序

### 全排序

- 过程:
	- 先进行排序,排序完取个中间值,作为分区的条件,就会保证是有序的

![][1]

抽样,求中值,根据中值定义partition




  [1]: https://www.github.com/wxdsunny/images/raw/master/1507942865699.jpg