---
title: Clojure快速排序
date: 2016-03-09 18:16:49
tags: [Clojure, 快速排序]
---

QQ: 380800878, 微信: kittenll

看过Haskell的快速排序之后，真的是被折服了，简洁明了。现在用Clojure也来写一个快排，同样也是相当的简单。

```clojuree
(defn quick-sort [coll]
  (if (empty? coll)
    []
    (let [[first & r] coll]
      (concat
        (quick-sort (filter #(< % first) r))
        [first]
        (quick-sort (filter #(>= % first) r))))))
```

算法的定义：排过序的 List 就是令所有小于等于头部的元素在先（它们已经排过了序）， 后跟大于头部的元素（它们同样已经排过了序）。
