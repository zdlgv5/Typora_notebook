# git使用教程

## 说明：

- 1.此教程适用于有一丢丢基础的git小白，至少已学完或者看完[廖雪峰git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)的前三章；

- 2.以xbot_pose仓库为例，模拟多人协作进行xbot_pose的开发和管理。

- 3.由于我还未参与过实际的协作开发流程，所以这里进行的测试主要来源于静姐那边遇到的问题，可能不是很全面，甚至有错误，希望大家可以给出建。

- 4.非最终版，有问题及时更正。

## 包含的知识点：

- 1.fork源仓库到自己的gitlab，并git clone到本地；

- 2.推送修改到远程仓库；

- 3.发起 merge request；

- 4.本地仓库与远程仓库同步、Github/gitlab进行fork后如何与原仓库同步；

- 5.解决代码合并中的冲突；

- 6.代码合并过程中rebase和merge的对比；

- 7.使用rebase合并多个commit为一个完整commit；

- 8.添加 ssh key 的相关操作（如不熟悉请优先阅读）

- 9.MR 后还需要本地修改时的操作（MR 后出现冲突提示请看 04.Github/gitlab进行fork后如何与原仓库同步）

假设原本的xbot_pose有两个分支，master和POSE_PC，master用于记录和发布给客户的最终版本，POSE_PC用于记录和发布内部测试版本。

假设原xbot_pose的管理者为jiuye，jiuye主要负责代码管理（代码合并以及承担部分开发任务）。假设协作开发者1号名叫wxy，另外还有若干协作开发者。协作开发者最终需要把自己的修改提交到POSE_PC上，jiuye整合成可用于测试的版本。

协作开发者wxy的开发流程如下：

- fork 原仓库到自己的gitlab中；

- git clone 后本地修改，push到gitlab；

- 发起merge request，如果冲突，解决；

管理者jiuye的任务：

- 判断是否同意其他协作者发来的merge request代码合并请求；

- 生成可用于测试的测试版本给测试组；

### 01.fork源仓库并git clone到本地

首先，需要fork 一下原仓库，fork命令在gitlab仓库右上角：

接着，把fork之后的仓库git clone到本地，可以执行`git remote -v`命令查看本地仓库对应的远程仓库：

当使用git clone命令时，会自动把本地的master分支和远程的master分支对应起来，远程仓库默认名字是origin。

上面显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址。

用`git branch`命令查看本地分支，可以看到只有master分支，并没有POSE_PC分支。但wxy是需要在POSE_PC上开发的，所以还必须创建远程origin的POSE_PC分支到本地，命令如下：`git checkout -b POSE_PC origin/POSE_PC `

此时git branch 可以看到已经有了POSE_PC分支，并且wxy此时正处于POSE_PC下。打开本地的xbot_pose目录也可以看到里面的内容和gitlab上仓库里的一致，下面，就可以愉快地在本地POSE_PC分支里进行开发了。

### 02.推送修改到远程仓库

举个例子，w做了很多开发和修改，可以经过以下几条命令把修改push到远程仓库的 POSE_PC分支：

- `git add -A`

- `git commit -m "commit"` 

- `git push origin 分支名`  

### 03.发起 merge request

w修改的内容push到远程仓库后，可以在w/xbot_pose仓库里看到对应的内容，现在，w需要把自己仓库的代码通过merge request与原仓库（jiuye管理）合并。选择gitlab上w的POSE_PC分支，右上角有发起MR的按钮，如下图：

title和description可以随便写。

注意选择对应的分支，最后提交MR，此后jiuye就会收到w提交的MR，jiuye同意了这次的MR后，原xbot_pose仓库下就有了w修改后提交的代码。

下面讲几个多人协作中常见的问题。

### 04.本地仓库与远程仓库同步、Github/gitlab进行fork后如何与原仓库同步

#### 本地仓库与远程仓库同步

jiuye本地有git clone 的xbot_pose与gitlab上xbot_pose 对应。此时jiuye在本地做了一些修改，push的时候发现有冲突，因为之前w对xbot_pose做了修改，jiuye本地的代码和gitlab上已经不一致。

 `git pull`进行同步后push通过：

#### Github/gitlab进行fork后如何与原仓库同步

w fork原仓库到自己的Gitlab中，过一段时间以后，原仓库可能会有各种提交以及修改，gitlab本身并没有自动进行同步的机制，所以w再次发起MR时，会出现合并冲突,点开Resolve conflicts可以看到详细的冲突信息，需要手动解决冲突（参考后面的解决冲突教程）

因此，我们需要确保自己本地的代码是原仓库的最新版本，参考[该教程](https://www.jianshu.com/p/57f05574a115)进行手动同步。

推送合并至远程的fork的仓库：`git push origin POSE_PC` 。实现fork后仓库和原仓库的同步。

### 05.解决代码合并中的冲突

在进行代码合并时，可能会提示有冲突：

根据提示的冲突文件，可以直接查看。

注意：git默认的文本编辑器是nano，执行下面的命令将git的文本编辑器改为我们熟悉的vim

```
git config --global core.editor vim
```

Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容，可以手动修改： 

修改后重新提交并查看分支合并情况： 

### 06.git rebase命令详解

它的作用简要概括为：可以对某一段线性提交历史进行编辑、删除、复制、粘贴；因此，合理使用rebase命令可以使我们的提交历史干净、简洁！

前提：不要通过rebase对任何已经提交到公共仓库中的commit进行修改（自己一个人玩的分支除外）。

#### 代码合并过程中rebase和merge的对比

假设w在本地仓库又对POSE_PC进行了修改，这里为了讲解git rebase，假设wxy在POSE_PC分支上新建了debug分支，此分支用于保存记录自己在本地的代码调试过程，当debug分支上的代码自己测试成功后，与POSE_PC合并，再将POSE_PC分支push到远程仓库，debug分支不push。 

创建debug分支并添加内容。 

回到POSE_PC分支，修改提交commit（为了测试冲突处理） 

再回到debug分支，进行修改提交 

总结： 

\- POSE_PC分支，节点链表指向为：0-2 

\- debug分支：0-1-3 

merge 合并与rebase的对比 

切换到POSE_PC分支，用git merge 命令进行合并。提示我们出现冲突：

手动修改后重新提交并查看分支合并情况：

看上去有点乱，那怎么办呢，可以选择rebase命令。 

跟merge 一样，提示有冲突，手动解决冲突后，git rebase --continue让rebase继续，查看历史就变成了一条线。 

总结：

- 代码合并过程中，使用rebase和merge命令合并后的结果没有任何区别，只是commit历史有区别。rebase命令的历史是一条线，比较简洁。

#### 使用rebase合并多个commit为一个完整commit

参考教程：[rebase用法小结](https://www.jianshu.com/p/4a8f4af4e803)

忘截图了，上面的教程很详细，可参考。

相关issue：[issue](https://yt.droid.ac.cn/beijing/weloveinterns/issues/324)

有疑问、建议和补充可到上述issue下讨论。

### 08. 在 Gitlab 上添加 ssh key

> 以下教程在 windows 系统上使用 git bash 操作，linux 系统请在用到 git bash 的地方换为 terminal

按照教程使用 git clone 之前，请先确认自己有没有添加 ssh key。可以按照下列步骤进行：

**Step1** 在右上方个人资料中点击 Settings

**Step2** 于侧边栏找到 SSH Keys

> 如果正确添加，会如下图所示，当然如果你没有添加过，此处应该为空。

**Step 3** 打开git bash，输入命令

```
ls -al ~/.ssh
```

检查是否显示有id_rsa.pub或者id_dsa.pub存在(下图为存在 .ssh 的示例)，如果存在请直接跳至第 5 步。

**Step 4** 使用以下命令，生成 id_rsa和id_rsa.pub文件，注意将 `your_email@example.com` 替换为自己的邮箱地址。***如下图所示，三个地方都可以直接回车。第一个地方提示存储位置，此处因为我已经有一个文件，因此更改了位置。第二个地方提示输入密码，可以不输入。第三个地方重复输入相同密码。***

```
ssh-keygen -t rsa -C "your_email@example.com"
```

**Step 5** 按照上一步的生成路径找到 `id_rsa.pub` 文件，用文本编辑器打开后复制全部内容（即 ssh key 信息）。

**Step 6** 按照 **Step 1** 到 **Step 3** 提示回到 **Step 3** 处的页面.

### 09. 提交 MR 后需要修改

有时在提交 MR 后会收到进一步的修改意见，此时在本地修改好后应该如何提交远程呢？

**先说结论：**

**Status 1**：在一次本地修改以后就可以提交的情况下，我们不需要更改 MR，只需要从本地进行 git push 操作，将本地代码与自己 fork 的远程仓库同步，MR 会自动更新。

**Status 2**：在多次本地修改、上传测试以后才最终提交的情况下，我们可以先关闭 MR。在最终版本确定后，通过 git push 操作，将本地代码与自己 fork 的远程仓库同步，再 reopen MR，此时 MR 也会自动更新（显示时与第一种情况稍有不同）。

下面看实例：新建了 test 仓库

在本地创建 dev 分支（后续 MR）并添加 test.txt 后，使用 git push 同步

提交 MR

假设此时本地 dev 分支添加 fix_afterMR.txt 文件后，直接 git push 与远程 dev 分支同步。查看 MR

**Status 1 演示完毕，下面演示 Stage 2**

\---

关闭 MR，并在本地 dev 分支添加 after_close.txt 与 after_close2.txt 文件后与远程 dev 分支同步

reopen MR 并查看

主页面没有显示版本更新提示，但是 commits 数量改变了，点击进行查看

证明本地改变都完美地记录在 MR 中。合并并查看 master 分支

### 10. 基本命令

- 使用命令：git clone 项目远程地址
- 使用命令：git branch //查看本地分支状态
- 使用命令：git branch -a //查看远程分支状态
- 使用命令：git checkout -b dev-test origin/dev-test //创建本地分支并跟踪远程分支（dev-test为分支名）
- 使用命令：git branch -vv //查看分支跟踪情况

### 其他命令

![选区_001.png](https://i.loli.net/2020/01/06/m9ralAdPtgxoKb4.png)

###### 参考链接

[Git常用命令](https://m.toutiaocdn.com/i6759348431216443911/?app=news_article&timestamp=1578056991&req_id=2020010321095001001404704024602BE9&group_id=6759348431216443911&wxshare_count=1&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_android&utm_campaign=client_share&from=singlemessage)