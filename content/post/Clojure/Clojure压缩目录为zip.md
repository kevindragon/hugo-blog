---
title: Clojure压缩目录为zip
date: 2018-03-08 13:37:04
tags: [Clojure]
---

创建zip压缩文件主要思想：
1. 创建ZipOutputStream
2. 创建ZipEntry，并调用putNextEntry
3. copy文件到output stream
4. 调用closeEntry关闭entry

<!-- more -->

```Clojure
(require '[clojure.java.io :as io]
         '[clojure.string :as str])
(import (java.io FileOutputStream)
        (java.util.zip ZipEntry ZipOutputStream))

(defn zip-dir
  "以zip格式压缩目录path"
  [path zip-file]
  (let [root-path (.toPath (io/file path))] 
    (with-open [zip (ZipOutputStream. (FileOutputStream. zip-file))]
      (doseq [f (file-seq (io/file path))]
        (let [p (.toPath f)
              subpath (str/replace (str (.relativize root-path p)) "\\" "/")
              path-name (if (.isDirectory f) (str subpath "/") subpath)]
          (when-not (= "" subpath)
            (.putNextEntry zip (ZipEntry. path-name))
            (when (.isFile f)
              (io/copy f zip))
            (.closeEntry zip)))))))
```

特别需要注意:
1. 是在windows系统下，文件路径是以反斜杠（\）分割的，需要转换成斜杠（/）。`(str/replace (str (.relativize root-path p)) "\\" "/")`
2. 如果是目录，entry需要以斜杠（/）结尾
3. 如果是要压缩的根目录，需要忽略

ZipEntry的构造函数为：`ZipEntry(String name)`，目录需要以/结尾

