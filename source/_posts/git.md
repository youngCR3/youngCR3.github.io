---
title: git
date: 2022-04-14 21:31:37
tags: git
categories: Tools
---

git常用命令

<!--more-->

# 基本命令

- commit：提交记录
- branch：创建新分支
- checkout：切换分支

# 合并分支

- merge：合并分支

- rebase：另一种合并分支

  ```bash
  git rebase side1 side2
  # 将分支side2合并到分支side1上
  git pull --rebase
  # 拉取远程仓库，并将本地分支合并到远程分支上
  ```

  

  

# 移动位置

- HEAD：指向某个分支或指向某个提交记录，指向提交记录时称为分离的HEAD
- 相对移动
  - 使用`^`向上移动1个记录，如`git checkout main^`会让HEAD移动到main分支的父节点
  - 使用`~<num>`向上移动多个提交记录，如`~3`

- 强制修改分支位置

  ```bash
  git branch -f main HEAD~3
  # 强制main分支指向HEAD的第3级父提交
  ```

# 撤销变更

- reset

  ```bash
  git reset HEAD^
  # 撤销本地分支
  ```

- revert

  ```bash
  git revert HEAD
  # 撤销远程分支，实际上会提交一个新纪录，其状态与HEAD的父节点状态一致
  ```

# 整理提交记录

- cherry-pick

```bash
git cherry-pick C3 C4 C7
# 将其他分支的某些提交添加到当前HEAD
```

- 交互式rebase

```bash
git rebase -i HEAD^^
```

# debug分支的技巧

- debug分支中，可将打印log的语句作为一个提交记录，debug的修改作为另一个提交记录，合并时直接cherry-pick debug修改对应的提交记录即可

# 标签

- tag

  ```bash
  git tag v1 C1
  # 为提交记录C1添加标签v1
  git tag v2
  # 如果不指定提交记录，则为默认值HEAD
  ```

- 寻找离我最近的标签`git describe`

  ```bash
  git describe <ref>
  # <ref>是任意能被git识别成提交记录的引用，默认值为HEAD
  # 输出结果是
  <tag>_<numCommits>_g<hash>
  # tag表示距离最近的标签，numCommits表示ref与tag相差多少个提交记录，hash表示ref所表示的提交记录的哈希值的前几位
  ```

  

# 多分支合并

# 移动到指定的父提交

- `^`可以移动到父节点
- `^2`可以移动到第二个父提交节点（其他数字以此类推）

## 移动支持链式操作

```bash
git checkout HEAD~^2~2
# 移动到HEAD的父节点的第二个父提交节点的往上两个父节点
```

# 远程分支

- git clone：拷贝远程分支到本地
- git commit导致分离的HEAD：在远程分支git commit会导致分离HEAD，因为本地修改并不会立刻同步到远程分支

- git fetch
  - 从远程仓库下载本地仓库缺失的提交记录，更新远程分支指针
  - 不会做的事：不会更新main分支，也不会修改磁盘上的文件，git fetch并不能让本地仓库与远程仓库同步
- git pull：相当于先执行git fetch，再执行git merge，实现本地仓库与远程仓库同步
- git push：将变更上传到指定的远程仓库，并在远程仓库上合并新提交记录

## 偏离的工作

- 偏离的含义：克隆远程仓库，并基于提交记录C1开发功能，形成提交记录C3。同时，合作者基于C1提交了C2，并推送到远程仓库。此时我无法直接push，因为远程仓库已更新到C2。Git强制我先合并远程仓库，再进行push

- 解决

  ```bash
  git fetch
  git rebase origin/master
  git push
  ```

## 远程服务器拒绝

- 如果你直接提交(commit)到本地main, 然后试图推送(push)修改, 你将会收到这样类似的信息:

  `! [远程服务器拒绝] main -> main (TF402455: 不允许推送(push)这个分支; 你必须使用pull request来更新这个分支.)`

  如果你是在一个大的合作团队中工作, 很可能是main被锁定了, 需要一些Pull Request流程来合并修改。

- 解决方法：先创建一个feature分支并push，reset main分支再push

  ```bash
  git checkout -b feature
  git push
  git checkout main
  git reset HEAD^
  git push
  ```

  

## merge与rebase的优缺点对比

- 优点：rebase使提交数更干净，所有提交都在一条直线上
- 缺点：rebase修改了提交树的历史。如下图，C2是比C3更早的提交记录，当使用rebase将side合并到main时，将使得C3成为C2的父节点，似乎C3比C2提交一样。

![](git_rebase.drawio.svg)

## 远程跟踪分支

- git中main分支与o/main是相关的

  pull操作时，提交记录先下载到o/main分支，再合并到本地的main分支

  push操作时，把工作从main推到远程仓库中的main分支，同时更新远程分支o/main

- **remote tracking属性**：main和o/main分支的关联关系是由分支的remote tracking属性决定的，main被设定为跟踪o/main，即main分支制定了推送的目的地以及拉取后合并的目标

- **git clone所做的事**：main分支的remote tracking属性无需我们手动设置，而是git clone时由git自动设置的。git为远程仓库中的每个分支在本地仓库中创建一个远程分支（如o/main），并创建一个跟踪远程仓库中活动分支的本地分支（如main）

  ```
  local branch "main" set to track remote branch "o/main"
  ```

- 手动设置remote tracking属性

  ```bash
  git checkout -b totallyNotMain o/main
  # 创建一个totallyNotMain分支，跟踪远程分支o/main
  git branch -u o/main foo
  # 让foo分支跟踪o/main
  ```

## git push的参数

- git push可以指定参数

  ```bash
  git push <remote> <place>
  git push origin main
  # 切换到本地仓库main分支，获取所有提交，并到远程仓库origin中找到main分支，将远程仓库中没有的提交记录都添加上去
  # main是<place>参数，代表git提交记录来自于main，要推送到远程仓库中的main，就是要同步的两个仓库的位置
  ```

- 本地仓库的分支与远程仓库的分支名字（即上述place）不同的情况

  ```bash
  git push origin <source>:<destination>
  # source和destination可以是不同名字的分支，事实上source只要是能被git识别的提交树位置即可，如HEAD^，而destination如果不存在于远程仓库将新建该分支
  git push origin HEAD^:main
  ```

  

## git fetch的参数

- 与git push的`<source>:<destination>`版本类似

  ```bash
  git fetch origin <source>:<destination>
  ```

## 省略source参数

- git fetch中省略source参数

  ```bash
  git fetch origin :<destination>
  git fetch :bar
  # fetch空到本地，创建bar分支
  ```

- git push省略source参数

  ```bash
  git push origin :<destination>
  git push origin :foo
  # push传空值source，删除远程仓库的foo分支
  ```

## git pull的参数

- git pull就是git fetch后跟git merge的缩写，因此git pull的参数可以理解为用同样的参数执行git fetch，再将当前checkout的位置merge所抓取的提交记录（如果提供了destination参数，则当前checkout的位置会merge到destination参数）

  ```bash
  git pull origin main:bar
  # 将远程仓库的main分支拉取到本地仓库的bar分支，同时将本地仓库当前checkout的位置merge到本地仓库的bar分支上
  ```

  
