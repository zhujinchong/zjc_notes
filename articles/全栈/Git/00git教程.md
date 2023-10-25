# window安装

从官网下载Git: https://git-scm.com/downloads 一直默认选项就可以。

安装完成后，打开git bash窗口，用命令修改配置文件，设置自己主机的名字和email

```
git config --global user.name "mingriyingying"
git config --global user.email "mingriyingying@163.com"
```

# 创建本地仓库

创建一个文件夹，在文件夹内打开git bash窗口，输入命令 `git init`，这个文件夹就变成了本地仓库。

本地仓库里面会有一个.git的隐藏文件夹，里面存储配置文件等内容。

# 本地仓库

## 提交与回退

> add表示从本地（工作区）提交到缓存中，commit表示从缓存提交到版本库。提交到版本库就可以记录历史版本。

提交到缓存区：`git add readme.txt`

提交所有文件到缓存：`git add --all`

提交到版本库：`git commit -m "xxxx"`

查看提交日志：`git log`

回退到上一个版本：`git reset --hard HEAD^`

回退到上上个版本：`git reset --hard HEAD^^`

回退到指定版本号：`git reset --hard 版本号前x位` （版本号通过`git log`查看）

查看提交日志：`git log`

查看最近10条提交日志：`git log -10` 

## 删除文件

第一步，删除工作区文件：`rm test.txt`

第二步，删除版本库文件：`git rm test.txt`，并提交：`git commit -m "remove test.txt"`

注意：

如果第一步操作失误，可以挽回，即撤销删除：`git checkout -- test.txt`

如果已经执行第二步，不能挽回。

# 远程仓库

## 本地仓库添加远程仓库

第一步，登录github，创建一个空仓库，如test。

第二步，在本地与远程仓库相连，远程仓库一般命名为origin：

`git remote add origin git@github.com:michaelliao/test.git`

第三步，将本地仓库推送到远程仓库：`git push -u origin master` （只有第一次需要-u参数）

第四步，以后本地作了提交，就直接推送到远程仓库：`git push origin master`

注意：

当你第一次使用Git的`clone`或者`push`命令连接GitHub时，会得到一个警告，输入yes回车就可以。

## 从远程仓库克隆到本地

SSH协议传输更快，但是有的公司不支持：

`git clone git@github.com:username/test02.git`

或者使用https协议传输：

`git clone https://github.com/username/test02.git`

# 分支

## 创建与合并分支

创建分支：`git branch dev`

切换到分支：`git switch dev`

查看当前分支：`git branch`

在当前分支完成内容并提交，并退回到master分支

合并分支：`git merge dev`

删除分支：`git branch -d dev`

## 分支管理策略

> 上述方式相当于，将master指针指向dev，在合并分支时使用的是Fast forward模式，删除分支后，分支信息会丢掉；
>
> 实际上，我们需要不断在分支做修改，然后master不断做合并，并记录合并的分支信息，此时需要使用另外一种合并模式

1. 在dev分支修改内容并提交
2. 切换到master
3. 合并分支：`git merge --no-ff -m "merge with no-ff" dev`

## Bug分支

> 情景：在dev分支开发时，master有个bug，需要到master修改；修改完毕后，回到dev，把master修改的bug复制下来，然后继续dev开发

1. 正在dev分支开发，master需要修改bug
2. 将dev开发内容”隐藏“：`git stash`
3. 切换到master
4. 在master修改bug，并提交，假设提交id为`1023ab`
5. 切换到dev
6. dev分支复制master的bug：`git cherry-pick 1023ab`
7. dev分支回复现场：`git stash pop`

## Feature分支

> 情景：软件新增一个功能，该功能还未发布，就已经不需要了，此时强制删除就可以

1. 在feature001分支上修改并提交
2. 切换到master
3. 删除分支：`git branch -d feature001`，提示还未合并，删除失败！
4. 强制删除：`git branch -D feature001`

## 多人协作

> 情景：多个人往远程仓库推送，修改同一内容是发生冲突

1. 前提：别人已经往远程仓库推送内容

2. 推送分支：`git push origin dev`，发现推送失败，有冲突

3. 先把远程分支拉下来：`git pull`

4. 如果pull失败，说明本地dev和远程dev没有关联，建立链接：

    `git branch --set-upstream-to=origin/dev dev`

5. 再pull：`git pull`，pull成功，但是和本地合并冲突，打开文件修改冲突。

6. 解决冲突后提交：`git commit -m "fix dev conflict"`

7. 推送：`git push origin dev`

## Rebase

> 情景：在我们pull别人提交的时候，本地原本直线的log变成了分叉，不美观，为了不让分叉可以使用rebase

1. 前提：别人已经往远程仓库推送内容
2. 自己本地完成提交
3. 先拉去别人内容：`git pull`
4. 如果有冲突解决冲突，并提交
5. 此时提交日志分叉，变成直线：`git rebase`
6. 推送

# 标签

> 标签像是指针，指向某个commit

打标签：

1. 切换到需要打标签的分支：`git switch master`
2. 打标签：`git tag v1.0`
3. 查看标签：`git tag`
4. 给之前提交打标签：`git tag v0.9 f52c633`
5. 打标签时添加注释：`git tag -a v0.8 -m "version 0.8 released" 109ad1`

删除标签：

1. 删除标签：`git tag -d v0.8`
2. 将标签推送到远程：`git push origin v1.0`
3. 将所有标签推送到远程：`git push origin --tags`
4. 删除远程之前，先删除本地：`git tag -d v1.0`
5. 再删除远程：`git push origin :refs/tags/v1.0`

# 自定义别名

> 命令太长记不住，可以自定义别名

带参数的日志，方便查看分支合并情况，解决分支冲突时可以使用：

```
git config --global alias.lg "log --graph --pretty=oneline --abbrev-commit"
```

# 其他

1. 中国Git托管Gitee（速度更快？）
2. 忽略特殊文件（避免上传特殊文件）
3. 搭建Git服务器
4. SourceTree可视化工具