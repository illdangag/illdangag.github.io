---
title: "lucene 간단 예제"
description: 'java와 lucene으로 텍스트 인덱싱'
author: illdangag

date: 2021-07-23T00:00:00+09:00
last_modified_at: 2021-07-23T00:00:00+09:00

categories: [Java]
tags: [Java, Lucene]
---

검색 엔진으로 많이 사용되고 있는 lucene에 대한 간단 예제이다  
프로젝트는 Gradle, 언어는 Java로 작성 하였다

## 프로젝트 설정

```groovy
dependencies {
    // https://mvnrepository.com/artifact/org.apache.lucene/lucene-core
    implementation group: 'org.apache.lucene', name: 'lucene-core', version: '8.9.0'
    // https://mvnrepository.com/artifact/org.apache.lucene/lucene-analyzers-common
    implementation group: 'org.apache.lucene', name: 'lucene-analyzers-common', version: '8.9.0'
}
```
build.gradle 파일의 dependencies에 lucene 라이브러리 두가지를 추가한다  
사용 가능한 버전의 목록은 [mvnrepository.com](https://mvnrepository.com/)에서 라이브러리 이름으로 검색한다

## 예제

```java
public class Person {
    private String id;
    private String name;
    private long age;
    public Person(String id, String name, long age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
    public String getId() {
        return id;
    }
    public String getName() {
        return name;
    }
    public long getAge() {
        return age;
    }
}
```
{: file='Person.java'}

위와 같은 데이터를 인덱싱

```java
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.LongPoint;
import org.apache.lucene.document.StringField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.Term;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import java.io.File;
import java.util.Arrays;
import java.util.List;

public class IndexMain {
    public static void main(String[] args) throws Exception {
        File indexDirectory = new File("index"); // 인덱싱 파일이 저장될 디렉토리 경로
        Directory directory = FSDirectory.open(indexDirectory.toPath());
        IndexWriter indexWriter = new IndexWriter(directory, new IndexWriterConfig(new StandardAnalyzer()));
        // 데이터 셈플 생성
        List<Person> personList = Arrays.asList(
                new Person("id0", "이름0", 30),
                new Person("id1", "이름1", 31),
                new Person("id2", "이름2", 32),
                new Person("id3", "이름3", 33),
                new Person("id4", "이름4", 34)
        );
        for (Person person : personList) {
            Term term = new Term("ID", person.getId());
            Document document = new Document();
            document.add(new StringField("id", person.getId(), Field.Store.YES));
            document.add(new StringField("name", person.getName(), Field.Store.NO));
            document.add(new LongPoint("age", person.getAge()));
            indexWriter.updateDocument(term, document);
        }
        indexWriter.commit();
    }
}
```
{: file='IndexMain.java'}
프로젝트 디렉토리에 index를 생성하고 위 코드를 실행하면 인덱스 파일이 생성 된다

```java
import org.apache.lucene.document.Document;
import org.apache.lucene.document.LongPoint;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.*;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import java.io.File;

public class SearchMain {
    public static void main(String[] args) throws Exception {
        File indexDirectory = new File("index");
        Directory directory = FSDirectory.open(indexDirectory.toPath());
        IndexSearcher indexSearcher = new IndexSearcher(DirectoryReader.open(directory));
        TermQuery nameQuery = new TermQuery(new Term("name", "이름1"));
        Query ageQuery = LongPoint.newSetQuery("age", 31);
        BooleanQuery.Builder booleanQueryBuilder = new BooleanQuery.Builder();
        booleanQueryBuilder.add(nameQuery, BooleanClause.Occur.MUST);
        booleanQueryBuilder.add(ageQuery, BooleanClause.Occur.MUST);
        TopDocs topDocs = indexSearcher.search(booleanQueryBuilder.build(), 10);
        System.out.println("count : " + topDocs.totalHits.value);
        long searchCount = topDocs.totalHits.value;
        for (int index = 0; index < searchCount; index++) {
            Document document = indexSearcher.doc(topDocs.scoreDocs[index].doc);
            System.out.println(" - id : " + document.get("id"));
        }
    }
}
```
{: file='SearchMain.java'}

```
count : 1
 - id : id1
```
{: file='output'}

인덱싱 결과에 대해서 `name`이 `이름1`이면서 `age`가 `31`인 문서를 조회 한다  
검색된 문서의 수와 문서의 id인 `id1`이 출력된다

조건이 AND가 아니고 OR이거나 NOT인 경우 `BooleanQuery`의 조합을 `BooleanClause.Occur.SHOULD`이거나  
`BooleanClause.Occur.MUST_NOT`으로 적절하게 조합하여 쿼리를 만든다

```java
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.Term;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import java.io.File;

public class UpdateMain {
    public static void main(String[] args) throws Exception {
        File indexDirectory = new File("index");
        Directory directory = FSDirectory.open(indexDirectory.toPath());
        IndexWriter indexWriter = new IndexWriter(directory, new IndexWriterConfig(new StandardAnalyzer()));
        Term term = new Term("id", "id1");
        indexWriter.deleteDocuments(term);
        indexWriter.commit();
    }
}
```
{: file='UpdateMain.java'}
```text
count : 0
```
{: file='output'}

생성된 인덱스에서 `id`가 `id1`인 문서를 삭제한다  
그 후에 검색 코드를 다시 실행하면 검색 결과가 0이다

## 참고 페이지
- [lucene.apache.org](https://lucene.apache.org/)
