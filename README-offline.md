## 一.文件结构
├── conf #配置文件所在目录

├── data #数据所在目录

├── include #头文件所在目录

├── Makefile #编译文件

├── README.md 

├── src #*.cpp文件所在目录


## 二.说明

本离线方式是将本地文件进行以下处理

### 2.1.格式化成一个网页库中data/ripepage.lib

+ 网页库的格式采用xml的格式，细分为

```html
<doc>
	<docid>1</docid><!--id号-->
	<url>https://averagjoe.wang</url><!--具体内容的索引，是文章在文件中的绝对路径-->
	<title>我的博客</title><!--文章标题，没有标题就默认提取文档第一行为标题-->
	<content>记录学习过程</content><!--文章的内容，包括标题-->
</doc>
```

### 2.2.在格式化成网页库中生成相应的偏移库data/offset.lib


### 2.3.网页去重


> 网页去重所使用的算法为TopK算法

+ 为每个网页提取网页中的特征码，特征码为该网页中词频（就是一个词在该网页中出现的次数）最高的10个词----top10词；
+ 对于每两篇网页，比较top10词的交集，如果交集大于6个，认为它们是互为重复的网页。在计算top词的时候，就需要使用分词程序，即将一篇网页分词，然后统计词频。
+ 互为重复的网页，删除哪一个呢？比如说，删除docid大的，或者删除文档内容少的。
+ 采用此方法，需要先用分词库对文档进行分词，然后去停用词，最后统计每篇文档的词语和词频。
+ 采用了[cppjieba](https://github.com/yanyiwu/cppjieba)中文分词库进行分词



### 2.4.生成新的data/ripepage.lib与偏移库data/offset.lib

### 2.5.建立倒排索引



> 倒排索引：英文原名为Inverted index，大概因为 Invert 有颠倒的意思，就被翻译成了倒排。一个未经处理的网页库中，一般是以文档ID作为索引，以文档内容作为记录。而Inverted index 指的是将单词或记录作为索引，将文档ID作为记录，这样便可以方便地通过单词或记录查找到其所在的文档。


在此项目中，倒排索引的数据结构采用的是:

```cpp
	unordered_map <string, vector<pair<int, double>>> InvertIndexTable
```

其中unordered_map的key为出现在文档中的某个词语，对应的value为包含该词语的文档ID的集合以及该词语的权重值w。

倒排索引的建立既是此项目的重点也是难点，而建倒排索引时最难的是每个词语的权重值的计算，它涉及到如下几个概念：

+ TF : Term Frequency, 某个词在文章中出现的次数；
+ DF: Document Frequency, 某个词在所有文章中出现的次数，即包含该词语的文档数量；
+ IDF: Inverse Document Frequency, 逆文档频率，表示该词对于该篇文章的重要性的一个系数，其计算公式为：

```
				IDF = log2(N/(DF+1))
```
其中N表示文档的总数或网页库的文档数


最后，词语的权重w则为：
```

				w = TF * IDF
```
可以看到权重系数与一个词在文档中的出现次数成正比，与该词在整个网页库中的出现次数成反比。

而一篇文档包含多个词语w1,w2,...,wn，还需要对这些词语的权重系数进行归一化处理，其计算公式如下：

```
			w' = w /sqrt(w1^2 + w2^2 +...+ wn^2)
```

w'才是需要保存下来的，即倒排索引的数据结构中InvertIndexTable的double类型所代表的值。此权重系数的算法称为TF-IDF算法。

+ 计算每一个次的归一化的权重，生成data/invertindex.lib

## 三.运行方法

### 3.1.修改

+ 将conf/my.conf中的文件路径修改成当前电脑的绝对路径

```cpp
yuliao			/home/joe/myGit/miniSearchEngine/offline/data/yuliao/
titlefeature	/home/joe/myGit/miniSearchEngine/offline/data/in.txt
ripepagelib		/home/joe/myGit/miniSearchEngine/offline/data/ripepage.lib
offsetlib	    /home/joe/myGit/miniSearchEngine/offline/data/offset.lib
stopword	    /home/joe/myGit/miniSearchEngine/offline/data/stop_words.utf8
newpagelib     /home/joe/myGit/miniSearchEngine/offline/data/newpage.lib
newoffsetlib    /home/joe/myGit/miniSearchEngine/offline/data/newoffset.lib
invertindexlib  /home/joe/myGit/miniSearchEngine/offline/data/invertindex.lib
```


+ 将include/WordSegmentation.h中的3个路径修改成当前目录下的绝对路径

```cpp
const char * const DICT_PATH = "/home/joe/myGit/miniSearchEngine/offline/data/jieba.dict.utf8";
const char * const HMM_PATH = "/home/joe/myGit/miniSearchEngine/offline/data/hmm_model.utf8";
const char * const USER_DICT_PATH = "/home/joe/myGit/miniSearchEngine/offline/data/user.dict.utf8";
```

### 3.2.运行

```bash
cd miniSearchEngine/offline
make
./main
#运行时间随电脑配置不一，一般在10分钟左右
```

