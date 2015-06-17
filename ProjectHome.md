# redis-search4j是一款基于redis的搜索组件。 #

## 特点 ##

1.基于redis，性能高效

2.实时更新索引

3.支持Suggest前缀、拼音查找(AutoComplete 功能)

4.支持单个或多个分词搜索

5.可根据字段进行结果排序

## 环境 ##

1.jdk 1.6+

2.redis 2.2+

## 依赖包 ##
1.Jedis-2.1.0

2.commons-pool-1.6.jar

3.IKAnalyzer-3.2.8.jar

4.pinyin4j-2.5.0.jar，已内置，无需添加

原理：参考 huacnlee的Rails App 运用 Redis 构建高性能的实时搜索，下载地址：http://code.google.com/p/redis-search4j/downloads/list


### 分词搜索 ###
将redis\_search\_config.properties添加到工程根目录下
添加相关依赖包：jedis，commons-pool，IKAnalyzer分词（创建索引时使用）

  1. 使用jar包内置的JedisPool获取jedis
```
        JedisHolder holder=JedisHolder.singleton();

        //JedisPool jp=holder.getJedisPoolInstance(String host,int port)
        //JedisPool jp=holder.getJedisPoolInstance(String host,int port,int timeout)
        //JedisPool jp=holder.getJedisPoolInstance(String host,int port,int timeout,String password)
        //JedisPool jp=holder.getJedisPoolInstance(String host,int port,int timeout,String password,int database)
        JedisPool jp=holder.getJedisPoolInstance("localhost");
        Jedis jedis=jp.getResource();
```

> 2.自己实现jedis
```
        Jedis jedis=new Jedis("localhost");
```
```
        
        //jedis.select(3);
        IndexWriter iw=new IndexWriter(jedis);

        //addIdAndIndexItem(id,"切分后的字符串，中间以“|”分隔");
        iw.addIdAndIndexItem("1","Ruby|on|Rails|为什么|什么|如此|高效");
	iw.addNeedSortItem("price","23.9");//需要排序的item
	iw.addNeedSortItem("date","2012");
	iw.addNeedSortItem("author","Klein");
	iw.writer();
	
	iw=new IndexWriter(jedis);
	
	iw.addIdAndIndexItem("2","Ruby|编程|入门|应该|看|什么");
	iw.addNeedSortItem("price","12.9");
	iw.addNeedSortItem("date","2011");
	iw.addNeedSortItem("author","Kevin");
	iw.writer();
	
	iw=new IndexWriter(jedis);
	
	iw.addIdAndIndexItem("3","Ruby|和|Python|什么|那个|更好");
	iw.addNeedSortItem("price","34.9");
	iw.addNeedSortItem("date","2009");
	iw.addNeedSortItem("author","Ben");
	iw.writer();
	
	iw=new IndexWriter(jedis);
	
	iw.addIdAndIndexItem("4","做|Rubies|开发|应该|用|什么|开发|工具|比较好");
	iw.addNeedSortItem("price","24.9");
	iw.addNeedSortItem("date","2012");
	iw.addNeedSortItem("author","Good");
	iw.writer();
	
	IndexSearch is=new IndexSearch(jedis);
	System.out.println(is.search("Ruby","什么"));
	System.out.println(is.search("price", IndexSearch.DESC, "Ruby","什么"));
	jp.returnResource(jedis);//将jedis放回pool中
         //redis.disconnect();
```

> 输出结果：
```
       [1, 2, 3]
       [3, 1, 2]
```
### Suggest前缀使用 ###
redis\_search\_config.properties中Suggest的配置

autoCompleteKey=ackey #AutoComplete集合的key

usePinyin=false       #是否开启拼音，内置pinyin4j包

maxRusult=10          #获取结果的数目

//添加词条
词条来源female-names.txt，下载地址： http://code.google.com/p/redis-search4j/downloads/list

```
Suggest s=new Suggest(jedis);
//读取female-names.txt文件进行添加
s.write(word);
```

//查找
```
SuggestSearch ss=new SuggestSearch(redis);
ss.search("be");
```
返回结果：

```
[bea, beatrice, beatrisa, beatrix, beatriz, bebe, becca, becka, becki, beckie]
```