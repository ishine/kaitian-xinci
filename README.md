# kaitian-xinci
开天-新词，中文新词发现与校对工具。

## 1 文本数据挖掘
[《互联网时代的社会语言学：基于SNS的文本数据挖掘》](http://www.matrix67.com/blog/archives/5044)，利用信息熵，直接从大规模的语料库中，自动发现可能成词的语言片段，从实际的效果看，还是非常不错。

前段时间，参考了上述文章，在一个领域提取新词，但发现效果不是很好，很多词虽然信息熵也很高，但看起意义不大，例如《西游记》的前20个词，如下：

```
Word,Length,Frequency,PMI,Entropy
与他,2,0.0006 ,3.1409 ,3.9583 
在此,2,0.0004 ,3.3108 ,3.9348 
唐僧,2,0.0013 ,5.2020 ,3.8754 
在那里,3,0.0002 ,3.8858 ,3.7310 
一个,2,0.0018 ,3.0593 ,3.7293 
有些,2,0.0003 ,3.4173 ,3.6657 
妖魔,2,0.0003 ,4.2062 ,3.6132 
甚么,2,0.0008 ,5.3953 ,3.6100 
这等,2,0.0004 ,3.5506 ,3.4867 
一条,2,0.0002 ,3.7682 ,3.4634 
菩萨,2,0.0011 ,6.7740 ,3.4491 
这般,2,0.0003 ,4.4282 ,3.4177 
天地,2,0.0001 ,3.0817 ,3.4134 
不能,2,0.0003 ,3.3489 ,3.4044 
老孙,2,0.0008 ,4.5552 ,3.4039 
不曾,2,0.0005 ,3.7950 ,3.3717 
在这里,3,0.0001 ,4.1741 ,3.3496 
几个,2,0.0002 ,3.3876 ,3.3452 
公主,2,0.0002 ,5.9290 ,3.3439 
吃了,2,0.0003 ,3.2121 ,3.3265 
```

比较明显的是“与他”，“在那里”，“在这里”，“吃了”，这些词，意义不大。

分析其中原因：
- 左右信息熵，例如“在”+“那里”也就是“在那里”，可能比“那里”的左信息熵还要大。

```
在那里,3,0.0002 ,3.8858 ,3.7310 
那里,2,0.0011 ,3.2877 ,2.9390
```

- 凝合程度，例如“在” + “那里”的聚合度，可能比“那” + “里”的聚合度还要大。

信息熵+凝合程度可以高效地刷选出候选新词，但候选新词是否有意义，需要通过其他方法进行二次刷选。

## 2 刷选有意义的新词
在自然语言中，词是由字组成，例如“在那里”，f(那里)=1是有意义的，f(在哪里)=0是没意义的，假设f()是判断某个候选词是否有意义的函数。

f()这个函数从哪里来？最快捷的方式是字典。假设字典里有“张三”这个词，但没有“李四”这个词，f()通过学习，知道 f(张三) = 1， 但 f(张三说) = 1，下次来了一个新句子包含“李四说”，那么，f()函数是否知道，f(李四) = 1， 而f(李四说) = 0？

我们通过学习 [bakeoff 2005](http://sighan.cs.uchicago.edu/bakeoff2005/) 中MSR训练的分词结果，例如 MSR训练集中有“云居寺”， f(云居寺) = 1，是否能够识别西游记中的“雷音寺” f(雷音寺) = 1?

```
信息熵+凝合程度，排在488位
雷音寺,3,0.0001 ,6.9698 ,1.8009
```

```
信息熵+凝合程度+f()，排在2位
雷音寺,3,0.0001 ,6.9716 ,1.8009 ,0.9995
```

《西游记》的前20个词，如下

```
Word,Length,Frequency,PMI,Entropy,Score
雷音寺,3,0.0001 ,6.9716 ,1.8009 ,0.9995 
孙大圣,3,0.0003 ,4.4028 ,2.9922 ,0.9994 
两个,2,0.0008 ,4.0659 ,3.0927 ,0.9993 
一件,2,0.0001 ,3.3204 ,3.2244 ,0.9992 
李天王,3,0.0001 ,5.5448 ,2.9134 ,0.9987 
几年,2,0.0001 ,4.0355 ,2.1627 ,0.9985 
几个,2,0.0002 ,3.3939 ,3.3319 ,0.9985 
当年,2,0.0001 ,4.3194 ,2.7271 ,0.9978 
五百年,3,0.0001 ,6.1414 ,1.1564 ,0.9977 
唐三藏,3,0.0001 ,3.1441 ,2.9921 ,0.9975 
不敢,2,0.0005 ,3.7793 ,3.0519 ,0.9975 
王母,2,0.0001 ,3.3968 ,2.8708 ,0.9973 
当时,2,0.0001 ,4.0423 ,2.3395 ,0.9969 
八戒沙僧,4,0.0001 ,3.6605 ,2.9422 ,0.9968 
下界,2,0.0001 ,4.0918 ,2.7696 ,0.9960 
不能,2,0.0003 ,3.3523 ,3.4194 ,0.9959 
法力,2,0.0001 ,4.9996 ,2.1978 ,0.9957 
谢恩,2,0.0001 ,5.3082 ,1.5721 ,0.9952 
上界,2,0.0001 ,3.4703 ,2.4323 ,0.9928 
大闹天宫,4,0.0001 ,4.6910 ,1.5734 ,0.9922 
```

能够有效地过滤掉类似“在这里”、“在那里”等意义不大的候选词。

这种方法拓展性比较强，如果觉得MSR的词汇量不够(约8.7万左右)，可以随时通过增加字典的方式，训练f()函数。

这种方法不是一点缺点也没有，例如 “取经”

```
取经,2,0.00036485779347448637,5.71485318922164,2.3223847703971217,0.4639596939086914
```

排到了468位，而在信息熵+凝合程度，排在301，经过分析，在MSR中，存在“争取经济稳定发展”，学到f(争取) = 1, 而 f(争取经) = 0，所以把 f(取经) 打的分数较低。

扣除字典里有的词(pkuseg-python带默认字典)，取概率>0.5的词，结果如下

```

信息熵+凝合程度+f()，前158个新词：

雷音寺, 孙大圣, 两个, 一件, 李天王, 几年, 几个, 当年, 五百年, 唐三藏, 不敢, 王母, 当时, 八戒沙僧, 下界, 不能, 法力, 谢恩, 上界, 大闹天宫, 芭蕉扇, 列位, 念动, 走出, 不识, 合掌, 在此, 美猴王, 十分欢喜, 孙行者, 牛魔王, 兄弟, 拜佛求经, 保唐僧, 呆子, 伯钦, 金睛, 揭谛, 闹天宫, 雷音, 施礼, 樵子, 耍子, 东土大唐, 祖师, 变作, 宝杖, 弼马温, 莫想, 西天取经, 真君, 四众, 那妖王, 金箍棒, 钵盂, 宝殿, 罗刹, 打死, 光蕊, 沙僧, 传旨, 星官, 猴王, 御弟, 层门, 洞门, 观音菩萨, 净瓶, 祥光, 慈悲, 木叉, 启奏, 驿丞, 千里, 驾云, 天兵, 牛王, 猢狲, 贫僧, 闻言, 铁棒, 灵霄, 龙王, 猪八戒, 关文, 壁厢, 驸马, 正果, 钉钯, 祸事, 毫毛, 降妖, 诸天, 伏侍, 心猿, 切莫, 雷公, 悟空, 打杀, 功曹, 妖魔, 妖邪, 万望, 那魔王, 孽畜, 怎生, 虽是, 原身, 山凹, 名唤, 比丘, 悟净, 天尊, 不肯, 听得, 老猪, 东土, 出城, 大圣, 怎敢, 身法, 妖精, 太宗, 赌斗, 女怪, 老君, 群妖, 八戒, 小龙, 大仙, 主公, 只因, 决不, 到此, 一只手, 国丈, 众僧, 老孙, 且休, 拜谢, 快早, 一片, 众神, 若要, 小妖, 二魔, 沙和尚, 多少, 我和你, 寡人, 必有, 成精, 特来, 老魔, 可曾, 众猴, 洞里, 小和尚, 天晚
```

共158个，其中‘保唐僧’、‘四众’、‘那妖王’、‘功曹’、‘那魔王’、‘快早’、‘我和你’、‘必有’、‘可曾’、‘天晚’可能意义不大，约占6%。
在所有344个新词中，概率<0.5的词, 做甚、悟能、上西天、阴司、那厮、取经人等可能有意义的词被遗漏掉，约占总词数的1.7%。效果感觉还可以。至少能省一部分人力进行人工校对。


```

信息熵+凝合程度，前158个新词：

与他, 在此, 在那里, 妖魔, 一条, 不能, 老孙, 在这里, 几个, 吃了, 妖精, 闻得, 孙行者, 一件, 不肯, 到此, 不识, 些儿, 大圣, 坐在, 猪八戒, 两个, 那厮, 一片, 不敢, 做个, 孙大圣, 唐三藏, 有甚, 小妖, 洞里, 拿住, 八戒沙僧, 天尊, 李天王, 全无, 一根, 王母, 美猴王, 老猪, 妖邪, 合掌, 遇着, 要吃, 伏侍, 虽是, 龙王, 降妖, 下界, 御弟, 有多少, 那妖精, 收了, 铁棒, 沙和尚, 当年, 木叉, 净瓶, 牛魔王, 一顿, 真君, 打死, 金箍棒, 观音菩萨, 取经的, 驸马, 洞外, 还未, 出城, 老君, 悟净, 被他, 多少, 丢了, 变作, 洞中, 八戒, 怎的, 驾云, 了多少, 沙僧, 唤做, 拿出, 悟能, 星官, 多官, 雷公, 众神, 用手, 也不曾, 有三个, 打杀, 怎生, 汝等, 猴王, 四众, 跳下, 有几, 诸天, 那厢, 钉钯, 兄弟, 弼马温, 走出, 教他, 殿上, 上界, 化斋, 洞门, 耍子, 取经人, 尽皆, 一场, 东土, 群妖, 迎着, 罗刹, 变做, 小和尚, 光蕊, 天兵, 端的, 与沙僧, 门首, 当时, 国丈, 一声, 扯住, 那女子, 拜谢, 贫僧, 众僧, 保唐僧, 大仙, 千里, 悟空, 揭谛, 听得, 那怪, 想是, 在旁, 误了, 在马上, 递与, 传旨, 太宗, 慈悲, 这厮, 伯钦, 我这里, 那道士, 个小妖, 法力, 猢狲, 那怪物, 这场, 快去, 莫要
```
共158个，其中‘与他’、‘在那里’，‘在这里’、‘吃了’、‘闻得’、‘些儿’、‘遇着’、‘要吃’、‘有多少’、‘那妖精’、‘收了’、‘取经的’、‘还未’、‘被他’、‘丢了’、‘了多少’、‘多官’、‘也不曾’、‘有三个’、‘迎着’、‘端的’、‘与沙僧’、‘门首’、‘那女子’、‘保唐僧’、‘听得’、‘那怪’、‘想是’、‘在旁’、‘误了’、‘在马上’、‘我这里’、‘那道士’、‘个小妖’、‘这场’可能意义不大，约占22%。

f() 算法在不断优化中......
 
## 3 参考
- [1] [互联网时代的社会语言学：基于SNS的文本数据挖掘](http://www.matrix67.com/blog/archives/5044)
- [2] [新词发现的信息熵方法与实现](https://kexue.fm/archives/3491)
- [3] [Chinese word segmentation algorithm without corpus（无需语料库的中文分词） ](https://github.com/Moonshile/ChineseWordSegmentation)
- [4] [新词发现算法(NewWordDetection) ](https://github.com/xylander23/New-Word-Detection)
- [5] ["新词发现"算法探讨与优化](https://zhuanlan.zhihu.com/p/80385615)
- [6] [kpot/keras-transformer](https://github.com/kpot/keras-transformer)
- [7] [GlassyWing/transformer-word-segmenter](https://github.com/GlassyWing/transformer-word-segmenter)


