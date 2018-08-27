<center>实战社工库搭建 solr架构 python开发查询页面</center>

## 0x前言

	之前就在坛子里和大家交流了下搭建裤子的问题,本来还以为估计要被喷死,没想到大家还是挺友好的,多谢理解。之前一直有ctf比赛 acm训练赛 之前本来和404约好8月初弄好的结果一下子到26号才开始弄,现在差不多了,后面就是导入数据和扩容 移植ssd的问题了(数据量大再过滤)



## 0x吐槽

elk的查询速度其实是比solr要好的 tools里面也有大佬分享了这个教程[elk搭建社工库](https://www.t00ls.net/viewthread.php?tid=32593&highlight=%E7%A4%BE%E5%B7%A5%E5%BA%93) 当时看了下 感觉硬盘要求有点高 而且是企业级的 后来在freebuf看到[技术分享：如何用Solr搭建大数据查询平台](http://www.freebuf.com/articles/database/100423.html) 这个文章 当时看到java tomcat 什么的 自己作为ctf一个web选手平时都没本地玩过 就耐着性子弄下去了。

## 0x正文

	环境:Mac pd虚拟机ubuntu16.04 安装mysql solr5.5.5 java8
	
	具体安装过程:参考上面的链接 
	
	其中有些细节参考下评论区来解决 (感觉solr还是low 不建议去安装 不过我已经入坑,分享经验)
	
	安装好后 做个优化
	
	solr开机自启动(方便做增量更新)
	
	按照上面那个教程 成功从mysql导入到solr中
	
	然后solr有web的查询接口:

```
http://10.211.55.6:8983/solr/solr_newsgk/select?q=keyword:xq17&start=10&rows=100&wt=json&indent=true
```

	返回的是json格式的数据 很方便
	
	然后我们只要本地写个查询web就行了 作者后面也给出了java和solr内置接口的代码 看起来比http要
	
	快.  不过我最近买了本<<flask开发>>比较薄的一本书还有bootstrap详解 然后干脆用这两个来锻炼下
	
	python开发能力。



![image-20180828005141969](https://ws2.sinaimg.cn/large/0069RVTdgy1fuoqtidmd8j31g20todoh.jpg)

ssbootstrap简单写了下模版:
![image-20180828005400458](https://ws2.sinaimg.cn/large/0069RVTdgy1fuoqvq453aj31kw0vq7wi.jpg)	
然后前端差不多弄好了,然后后台对key字段做了一些处理,比如对非纯数字字符加*达到精确查询的效
果。

##0x 导入数据 清理数据

	这里我采用某盟89w会员数据作为测试。 后期就是清理各种数据 导入的问题了 诚请教大佬们技巧

	首先得到的是一个sql数据. 我直接在naviat 转储数据结构 然后把sql还原 然后再导出向导 把id字段去掉

	这个时候就得到了一个比较规则的insert语句了

	然后把本来那个cdb_uc_member字段替换为````b41new`(username,password,email,salt)```

	然后再navicat sgk这个数据库那里右键运行sql文件 直接导入89w数据 然后sql update  添加

	来源 (ps:感觉自己做的有点多余,有数据清理导入数据的大佬求指点)

![image-20180828010724012](https://ws3.sinaimg.cn/large/0069RVTdgy1fuorczoi4nj30m80a8abr.jpg)

	做完这一步之后,接下来就是全量导入数据的问题了

![image-20180828011039606](https://ws3.sinaimg.cn/large/0069RVTdgy1fuorcws3l9j315805wjw5.jpg)

	接近90w条 导入花费了1min 42秒 这个速度的话导入1亿需要2*100=200/60=3.3小时

	要是用固态会好很多估计 虚拟机感觉跑的有点吃力 solr只要存好配置 很简单就可以在移植到另外

	一个平台 等数据量大了 我就去听画船师傅那样去搞个树莓派 弄个流弊的ssd 大出血。

	

## 0x效果
![image-20180828011456646](https://ws1.sinaimg.cn/large/0069RVTdgy1fuorhibg54j31kw0zykjl.jpg)

	这个0秒其实没那么快 是因为做了缓存的缘故 第二次才会那么快 一般都是几秒.

	这里我分享下数据库结构

```
`id` bigint NOT NULL auto_increment,
username varchar(255),
email varchar(255),
password varchar(255),
salt varchar(255),
ip varchar(255),
iphone varchar(255),
wechat varchar(255),
idno varchar(255),
realname varchar(255),
site varchar(255),
updatetime timestamp NOT NULL ON UPDATE CURRENT_TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
PRIMARY KEY (id)
```

	因为手头有一些校花的数据 微信 手机号 真实姓名就把这个字段弄进去 个人感觉没啥用 浪费前端位置

	后期考虑加个隐藏窗口(目的:学习bootstrap的js技巧啦) 美女啥的不在乎

## 0x增量更新

	增量更新solr很简单参考上文文章和网上的教程

	上面我也给出了updatetime这个字段	

	按照文章配置:

	db-data-config.xml

	这个文件就行了

	后面就可以选择直接get更新或者

	自己手动去(我就是手动的哈 自动啥的不存在的我电脑 又不是挂机的,树莓派后期考虑下)

![image-20180828012543778](https://ws2.sinaimg.cn/large/0069RVTdgy1fuorssh12vj31fg0riq6m.jpg)

	这里我的表没有更新数据 所以就没数据可以进行索引。

	

## 0xEnd

	这个东西是个长期维护的过程,清理数据和整理数据导入,还有对solr进行优化 达到更快的要求 都是我

	接下来空闲的时候去思考的。

	后面对小七论坛 还有 网易x箱 进行 数据清洗 这个挑战有点大 合起来差不多100g了

	弄好的话maybe可以分享下清理的思路(本菜鸡都是切割出来丢服务器慢慢跑脚本哈哈 50g硬盘)

## 0xTODO

	吗的。为啥我没入elk的坑呢 我是土豪啊。。。。。。

	算了还是十几万的车,我适合开

	ps:我感觉solr的搭建过程还是比较简单的 几个小时就能从一无所知到修修改改	

	由于web查询页面写的太垃圾 避免被同学嘲笑 故不开源 等后期 弄清楚flask开发 完善下再考虑丢

	githud。

	

	tools论坛好像不支持外链(图片挂的话请直接访问githud):	
