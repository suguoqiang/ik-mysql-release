热更新
=================================

每次都是在es的扩展词典中，手动添加新词语，很坑
（1）每次添加完，都要重启es才能生效，非常麻烦
（2）es是分布式的，可能有数百个节点，你不能每次都一个一个节点上面去修改

es不停机，直接我们在外部某个地方添加新的词语，es中立即热加载到这些新词语

热更新的方案

（1）修改ik分词器源码，然后手动支持从mysql中每隔一定时间，自动加载新的词库
（2）基于ik分词器原生支持的热更新方案，部署一个web服务器，提供一个http接口，通过modified和tag两个http响应头，来提供词语的热更新

用第一种方案，第二种，ik git社区官方都不建议采用，觉得不太稳定

1、下载源码

https://github.com/medcl/elasticsearch-analysis-ik/tree/v5.2.0

ik分词器，是个标准的java maven工程，直接导入eclipse就可以看到源码

2、修改源码
新增 HotDictReloadThread类（时间循环函数）：不断调用Dictionary.getSingleton().reLoadMainDict()，去重新加载词典
在 Dictionary类中，在原方法 loadMainDict 函数上，增加 this.loadMySQLExtDict() 方法; 在原方法 loadStopWordDict 函数上，增加 this.loadMySQLStopwordDict()方法;

3、mvn package打包代码

target\releases\elasticsearch-analysis-ik-5.2.0.zip

4、解压缩ik压缩包

将mysql驱动jar，放入ik的目录下

5、修改jdbc相关配置

6、重启es

观察日志，日志中就会显示我们打印的那些东西，比如加载了什么配置，加载了什么词语，什么停用词

7、在mysql中添加词库与停用词

8、分词实验，验证热更新生效
