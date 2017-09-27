---
title: Linux 
tags: linux,java
grammar_cjkRuby: true
---

This is linux 

``` java
@Test
public void test01() throws Exception {
	FSDirectory dir = FSDirectory.open(new File("d:/index"));
	Analyzer analyzer = new StandardAnalyzer();
	IndexWriterConfig config = new IndexWriterConfig(Version.LATEST, analyzer);
	IndexWriter iw = new IndexWriter(dir,config);
	Document doc = new Document();
	File file = new File("E:/Develop/lucene/source");
	File[] listFiles = file.listFiles();
	for (File file2 : listFiles) {
		String fileName = file2.getName();
		String filePath = file2.getAbsolutePath();
	    String fileContent = FileUtils.readFileToString(file2);
		long fileSize = FileUtils.sizeOf(file2);
		Field fieldName = new TextField("fileName", fileName,Store.YES);
		Field fieldContent = new TextField("fileContent", fileContent,Store.YES);
		Field fieldPath = new StoredField("filePath", filePath);
		Field fieldSize = new LongField("fileSize", fileSize, Store.YES);
		doc.add(fieldName);
		doc.add(fieldContent);
		doc.add(fieldPath);
		doc.add(fieldSize);
		iw.addDocument(doc);
	}
	iw.close();
}
```



[百度一下,你就知道][1]


  [1]: www.baidu.com