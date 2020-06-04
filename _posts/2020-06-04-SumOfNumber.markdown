---
layout:     post
title:      "自然数的和"
subtitle:   "Hello World, Hello Blog"
date:       2020-06-04
author:     "Caiiiiii"
header-img: "img/post.jpg"
tags:
    - 算法
    - 基础
---

#求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

```
public class Sum_Solution {
    public int Sum(int n){
        int res = n;
        // && 有短路求值的原理，在前一个为 false 的时候，
        //就不会执行后面一个，就能解决递归时当n<0的时候做不了判断的问题
        boolean flag = (n>0)&&((res+=Sum(n-1))>0);
        return res;
    }
}
```

