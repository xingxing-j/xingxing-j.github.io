---
title: Hexo多电脑同步及未解决的坑
date: 2020-04-21 15:33:32
categories: 博客
tags:
- [Hexo]
- [同步]
- [坑]
---

​	同步Hexo博客时遇到的各种奇葩的问题，当然最后还是没有搞懂啦(看懂是看不懂的，百度又百度不到=_=，就只能将就的样子)，只是记录下这些**莫名奇妙的坑**和**网上搜罗的同步步骤**。

<!-- more -->

​	之前顺利的部署了Hexo博客之后，就在想之后要是在不同设备上难不成还要再按网上步骤部署一遍，自定义主题再配置一遍，文档啥的，还要再手动`ctrl+c`和`ctrl+v`一次？那也太麻烦了吧？

​	不出所料，网上果然有同步设备之间进行Hexo博客同步的教程。只可惜真的是太乱了，不是你抄我，就是我抄你，其实这还不是最主要的，能解决问题就好了。可惜不是那么的顺利，真是一步一个坑。那些坑，我现在都满头的问号，所以这篇博客就记录下这些莫名的坑和我之后在网上找的看起来比较靠谱，自己试过且没报错的教程。

​	网上关于同步不同设备之间的Hexo博客的教程，基本上就是将Hexo博客根目录下的**源文件夹备份**到静态博客部网站(我是部署到了github上)的另外一个分支上。之后在其他设备上进行转移的时候，把源文件夹再拷贝下来，再进行简单的配置即可。

​	具体怎么**备份**，具体怎么再拷贝下来，网上的内容纷繁复杂，我挑了几个觉得看起来比较靠谱的：

- 首先是这位仁兄的博客[大专栏](https://www.dazhuanlan.com/2020/02/03/5e36f70fc57fd/)，发布时间为2020年2月份，虽然里面删除`.git`文件夹的命令写错了，但无伤大雅，能让我了解大概的流程。

- 然后是CSDN上的一片博客[最安全的 hexo 多电脑同步博客解决方案--非新建分支](https://blog.csdn.net/qq_35684085/article/details/85767516)，19年的，这篇博客在网上都不知道被转了几手了，我找的这篇也不知道是不是原创...=_=，该篇博客对上一篇博客的一些内容进行了补充，里面提到的一些建议看起来挺完善，但我并没有进行过试验，或者说还没有遇到过该篇里提到的问题，也就无从试验。

- 再然后是这篇，[在多台电脑间使用hexo](https://theqwang.github.io/2017/03/17/%E5%9C%A8%E5%A4%9A%E5%8F%B0%E7%94%B5%E8%84%91%E9%97%B4%E4%BD%BF%E7%94%A8hexo/#more)，本来我是打算参照这篇进行备份的。无奈，这位仁兄在“将源文件备份到github上的其他分支”的步骤上，一笔带过，萌新真是看不懂啊，于是就在`git push`上卡住了，掉进了一个令人万分痛苦的坑里，该坑至今未看懂，百度也摸不到头脑，正是该坑让我想要写这篇博客进行记录，说不定哪年哪月，再看到这篇博客，再去探寻就又有想不到收获呢？(等哪年吧，我现在想起那个坑便一阵犯恶心..)

- 最后是简书上的一篇[Hexo博客备份](https://www.jianshu.com/p/57b5a384f234)，17年的，我接下来的博客备份也是大部分参照的该篇博客(因为就它提供的步骤没有那些奇奇怪怪的问题—_—)。

   ​

   大致絮叨完后，让我们来看看，到底哪些是Hexo博客的源文件夹呢？下面会对Hexo博客根目录下的各个文件夹进行简单说明，以便了解哪些是Hexo的**源文件夹**：

   ​

- **_config.yml**：该文件是站点的配置文件，是**需要备份到github上的，以便换设备时进行拷贝**。

- **themes**：该文件夹是主题文件夹，里面放的是Hexo的默认主题文件资源，自定义的主题文件也是放到该文夹下的，**需要备份和拷贝**。

- **source**：存放博客文章的.md文件，**需要备份和拷贝**。

- **scaffolds**：是博客文章的模板，**需要备份和拷贝**。

- **package.json**：里面写有安装包的名称，`npm install`就是通过该文件里记录的来安装相关依赖，**需要备份和拷贝**。

- **package-lock.json**：这是个啥文件我也不清楚，网上教程大都没有提及该文件，或许是新内容？反正也不大，拷上就完事了，**需要备份和拷贝**。

- **.gitignore** ：限定在push时哪些文件可以忽略，**需要备份和拷贝**。

  ```txt
  .DS_Store
  Thumbs.db
  db.json
  *.log
  node_modules/
  public/
  .deploy*/
  ```

- **.git**：主题和站点都有，标志这是一个git项目，不需要备份和拷贝。

- **node_modules**：是安装包的目录，在执行`npm install`的时候会重新生成，不需要备份和拷贝。

- **public**：是`hexo g`生成的静态网页，不需要备份和拷贝。

- **.deploy_git**：同上，`hexo g`也会生成，其实仔细看，就会发现该文件夹里的文件跟我们部署到github的master分支上的内容一模一样，不需要备份和拷贝。

- `db.json`：不需要备份和拷贝。

  ​



​	接下来，就要开始进行**源文件**的备份了。实现多设备同步的思想就是用github的**master分支存放静态博客资源**，用**hexo分支存放源文件**，当我们更换新设备时，只需`git clone xxxxx`下来，再`npm install`和`npm install hexo-deployer-git`，就大体成了，大概。

​	起先我是想将原先的Hexo博客根目录里初始化成git目录，再将其下的**源文件**上传到github的hexo分支上的，当我输入下列命令时

```shell
$ git push github hexo hexo
```

​	就报出了令我恶心的坑了

```shell
error: dst ref refs/heads/hexo_source receives from more than one src
error: failed to push some refs to 'git@github.com:xingxing-j/xingxing-j.github.io.git'
```

​	百度嘛，也是各有各的说法，貌似都跟远程仓库上一个叫README.md的文件有关，我寻思我远程仓库也没有README.md呀，难不成非得有？删掉远程仓库，重建，这次添加README.md，重新关联，重新push，不行，还是一样的错误。难不成要先从上面拉取一下？pull完，再push，不行，还是报同样的错....难不成不能用pull，要fetch再reset？试一下，再pull，好嘛，又报了新错误

```shell
kex_exchange_identification: read: Connection reset by peer
```

​	这又是什么东西...不知名的ssh错误，我佛了，倒腾了一通宵，卡在第一步的push上，只能放弃在Hexo根目录下push**源文件**的想法。



下面来梳理我即将试验的步骤

### 0. 新建远程仓库

​	把原先github上的博客仓库删掉，新建一个**一模一样的空仓库**，就跟部署博客时要新建的仓库步骤一样，这里便不再赘述，网上教程很多。建完后，如果没有手动建README.md，在**本地初始化一个git目录** ，再新建README.md，按github的图示，将该目录与远程仓库关联，进行第一次的`git add .`、`git commit -m 'message'`、`git push -u 远程仓库名称 master`，这里应该是能push上去的。

​	搞这么多的目的就是要让远程仓库的master分支出现，我们才能整出远程仓库的hexo分支。远程仓库建好hexo后，就可以进行clone了，注意哦，此时远程仓库里，不管是master分支还是hexo分支，就只有个README.md哦。

### 1. 克隆hexo分支

​	接着，在原先的文件夹也好，新建文件夹也好，右键`Git Bash`，调出命令框，`git clone 远程仓库地址`克隆hexo分支，这里的clone命令，不需要在Git目录下也能执行哦，我才是才知道+_+。

### 2. 转移源文件夹

​	把之前博客根目录下的源文件复制到刚才clone好的文件夹里，**_config.yml**、**themes**、**source**、**scaffolds**、**package.json**、**package-lock.json**和**.gitignore** 。

​	网上的教程还提到了themes/next里.git文件夹，需要拿出来，可我的没有，只有个.github文件夹，也就顺手拿出来了。

### 3. 安装插件 

​	还是那个文件所有，执行`npm install`和`npm install hexo-deployer-git`或`cnpm install`和`cnpm install hexo-deployer-git`（这里可以看一看分支是不是显示为hexo）

### 4. 备份源文件到hexo分支上

​	我严重怀疑我git相关命令不熟，连怎么push,怎么pull都不清楚，所有一有问题就歇菜。

​	**git push 吧!!!**经过上面的两步操作后，再执行`git add .`、`git commit -m ""`，

最后`git push 远程仓库名称 hexo hexo`应该就能push上去了吧。

### 5. 部署静态资源

​	执行`hexo g && hexo d`生成静态网页部署至Github上。

至此，源文件和静态资源都备份到github上了。



### 换设备时的操作步骤

- 安装git
- 安装Nodejs和npm
- 使用`git clone 远程仓库地址`将仓库拷贝至本地
- 在文件夹内执行以下命令`npm install hexo-cli -g`、`npm install`、`npm install hexo-deployer-git`或`cnpm install hexo-cli -g`、`cnpm install`、`cnpm install hexo-deployer-git`。





### 修改时的操作

​	**每当修改博客（包括修改主题样式、发布新文章等）前**：

```shell
git fetch --all #将git上所有文件拉取到本地
git reset --hard origin/hexo  #强制将本地内容指向刚刚同步git云端内容
```

​	或执行`git pull`， 此步到底行不行还有待验证。



​	**每当修改博客（包括修改主题样式、发布新文章等）后**：

1. 依次执行`git add .`、`git commit -m ""`、`git push 远程仓库名称 hexo`来提交hexo网站源文件；
2. 执行`hexo g && hexo d`生成静态网页部署至Github上。


