---
layout: post
title: "ACM June Records"
description: "Monthly reports."
date: 2019-06-06
tags: [ACM, Algorithms, Interactive]
comments: true
---

我计划从后往前刷 CF Education Round，目前已结束的最新的一场是 [Round 65 ](http://codeforces.com/contest/1167)。

# [Educational Codeforces Round 65 ](http://codeforces.com/contest/1167)

## A. Telephone Number

给定字符串 s，问是否存在 s[i] == '8' && s.size() - i >= 11。

[Source code.](https://github.com/NeapolitanIcecream/Code/blob/master/cf1167/a.cpp)

## [Interactive] B. Lost Numbers

存在 4, 8, 15, 16, 23, 42 的一个排列 a，每次询问 (i, j)，返回 $a_i*a_j$ ，要求在至多四次询问内给出这个排列。

除了 4 * 16 = 8 * 8 以外，所有返回值只出现一次，所以：

(1) 每当询问两个互不相同的数，可以确定两个数分别存在于两个位置中的某一个；  
(2) 每当询问包括一个数，这个数是 (1) 中确定的，这两次询问中的三个数的位置可以确定。

应用两次 (1)(2)，可以在四次询问后确定六个数的位置。

[Source code.](https://github.com/NeapolitanIcecream/Code/blob/master/cf1167/b.cpp)

