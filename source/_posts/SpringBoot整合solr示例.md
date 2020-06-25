---
title: SpringBoot整合solr示例
date: 2020-06-25 15:03:31
categories: 后端学习
tags:
- [SpringBoot]
- [solr]
- [文件检索]
---

SpringBoot整合solr的一个示例

<!-- more -->

### Controller层

```java
@RestController
@RequestMapping("/game")
public class GameController {
	@Autowired
	IGameService iGameService;
	@GetMapping("/add/{name}/{size}")
	public String addGame(@PathVariable String name, @PathVariable Integer size) {
		iGameService.addGame(name, size);
		return "success";
	}

	@GetMapping("/query/{word}")
	public List queryGame(@PathVariable String word) {
		SolrDocumentList list = iGameService.searchGame(word);
		return list;
	}
}
```

### Service层

```java
public interface IGameService {

	void addGame(String name, Integer size);

	SolrDocumentList searchGame(String word);
}
```

```java
@Service
public class GameServiceImpl implements IGameService {
	@Resource
	private SolrClient solrClient;
	@Override
	public void addGame(String name, Integer size) {
		// 声明solr document对象,用来存储数据,并发送给solr
		SolrInputDocument document = new SolrInputDocument();
		// 存储数据 到 文档对象，以key-value形式存储
		document.addField("game_name", name);
		document.addField("game_size", size);

		try {
			// 存入solr              core的名字  数据
			solrClient.add("javasm", document);
			// 提交
			solrClient.commit("javasm");
		} catch (SolrServerException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@Override
	public SolrDocumentList searchGame(String word) {
		// 创建Solr查询对象
		SolrQuery query = new SolrQuery();
		// 设置检索条件
		query.setQuery(word);

		QueryResponse response = null;
		try {
			response = solrClient.query("javasm", query);
			// 获得返回的数据对象
			SolrDocumentList results = response.getResults();
			results.forEach(solrDocument -> {
				System.out.println(solrDocument);
			});
			return results;
		} catch (SolrServerException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}

		return null;
	}
}
```

