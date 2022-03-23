---
title: Lucene7实践入坑第一步
date: 2017-08-27 17:36:56
tags: [Lucene]
---

这是一篇很简单的使用Lucene7.0.0来建立索引和搜索的文章。

```java
import org.apache.lucene.analysis.core.WhitespaceAnalyzer;
import org.apache.lucene.document.*;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.PhraseQuery;
import org.apache.lucene.search.SearcherManager;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.FSDirectory;

import java.io.IOException;
import java.nio.file.Paths;

public class UseLucene2 {

    public static void main(String[] args) throws IOException {
        FSDirectory indexDir = FSDirectory.open(Paths.get("index"));
        IndexWriterConfig iwc = new IndexWriterConfig(new WhitespaceAnalyzer());
        iwc.setOpenMode(IndexWriterConfig.OpenMode.CREATE);
        IndexWriter indexWriter = new IndexWriter(indexDir, iwc);
        DirectoryReader dir = DirectoryReader.open(indexWriter);

        Document doc = new Document();
        doc.add(new StringField("first", "测试", Field.Store.YES));
        indexWriter.addDocument(doc);


        SearcherManager sm = new SearcherManager(indexWriter, true, true, null);


        PhraseQuery.Builder builder = new PhraseQuery.Builder();
        builder.add(new Term("first", "测试"));
        PhraseQuery pq = builder.build();

        IndexSearcher searcher = sm.acquire();
        TopDocs docs = searcher.search(pq, 1);

        System.out.println(docs.totalHits);
        for (int i = 0; i < docs.totalHits; i++) {
            System.out.println(docs.scoreDocs[i]);
        }

        sm.close();
        indexWriter.close();
    }

}
```

// TODO 补充解释
